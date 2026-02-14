# Project Technical Workflow Summary

> **Note:** The repository content available locally is a full-stack banking application (`bank-final` / `iobank`) rather than a credit-card fraud detection ML pipeline. This document summarizes the implementation and runtime workflow by analyzing the backend and frontend source folders in this project.

## 1) Implementation Overview

This project is implemented as a **two-tier full-stack system**:

- **Backend**: Spring Boot application exposing REST APIs for authentication, account operations, card operations, transactions, and currency conversion.
- **Frontend**: React + Redux SPA that authenticates users, stores JWT in session storage, and drives account/card/payment workflows through API calls.

At a high level, user actions in the UI dispatch Redux thunks → call backend endpoints with JWT headers → backend validates JWT and principal → services/helpers execute business logic → entities persist through Spring Data JPA repositories.

---

## 2) Repository & Module Layout

### Backend module (`/backend`)

Primary layers:

1. **Controller layer** (`controller/`): API contract and request routing.
2. **Service layer** (`service/`): business operations and orchestration.
3. **Helper layer** (`service/helper/`): focused account validation/transfer/conversion rules.
4. **Persistence layer** (`repository/`): JPA query access.
5. **Domain layer** (`entity/`, `dto/`, enums): core data model.
6. **Security layer** (`config/`, `filters/`): JWT and Spring Security chain.

### Frontend module (`/frontend`)

Primary layers:

1. **Routing and app shell** (`App.js`, `pages/`): public/authenticated route boundaries.
2. **Feature state** (`features/*Slice.js`): async API calls and normalized UI state updates.
3. **Page components** (`pages/dashboard/*`): user workflows (accounts, card, transfers, conversion, transactions).
4. **Reusable UI components** (`components/`): form controls, nav/header, card widgets.
5. **HTTP abstraction** (`api/api.js`): configured axios instance and request helpers.

---

## 3) Backend Runtime Workflow

## 3.1 Application startup

1. Spring Boot initializes the application context and component graph.
2. `AppConfig` wires key beans (authentication manager/provider, user details service, password encoder, `RestTemplate`, scheduler).
3. `SecurityConfig` builds a stateless filter chain and permits public auth endpoints while protecting all others.
4. `ExchangeRateScheduleTaskRunnerComponent` starts a scheduled job to refresh exchange rates every 12 hours.

## 3.2 Security and authentication flow

### Registration flow

- `POST /user/register` receives a `UserDto`.
- Service maps DTO to `User`, hashes password via `PasswordEncoder`, sets default role/tag, and persists via `UserRepository`.

### Login flow

- `POST /user/auth` authenticates credentials via `AuthenticationManager`.
- `JwtService` generates a signed JWT with subject + expiration.
- Controller returns user payload and exposes JWT in the `Authorization` response header.

### Protected request flow

- Frontend sends `Authorization: Bearer <token>`.
- `JwtAuthenticationFilter` validates and parses token, loads the user, and populates `SecurityContext`.
- Controller methods consume `Authentication` principal and pass user context to services.

---

## 4) Core Banking Domain Workflows

## 4.1 Account creation

- Endpoint: `POST /accounts`
- Workflow:
  1. Validate account type uniqueness per user.
  2. Generate unique account number.
  3. Build account with default opening balance (1000) and currency metadata.
  4. Persist account.

## 4.2 Funds transfer

- Endpoint: `POST /accounts/transfer`
- Workflow:
  1. Resolve sender account by currency code and authenticated user.
  2. Resolve recipient by account number.
  3. Validate sufficient funds including transfer fee (`amount * 1.01`).
  4. Debit sender, credit recipient.
  5. Save both accounts atomically.
  6. Create two transaction records (sender withdraw with fee, receiver deposit).

## 4.3 Currency conversion

- Endpoint: `POST /accounts/convert`
- Workflow:
  1. Validate amount > 0, source/target currency difference, ownership of both accounts, and source balance.
  2. Retrieve in-memory FX rates from `ExchangeRateService`.
  3. Compute converted value using `(toRate / fromRate) * amount`.
  4. Debit source (`+1% fee`) and credit destination.
  5. Save accounts + write conversion/deposit transactions.

## 4.4 Card operations

- Endpoints:
  - `GET /card`
  - `POST /card/create?amount=...`
  - `POST /card/credit?amount=...`
  - `POST /card/debit?amount=...`

### Card creation

- Requires minimum amount and an existing USD account.
- Debits USD account.
- Generates unique card number and CVV.
- Initializes card balance as `amount - 1` (implicit setup charge behavior).
- Persists card and writes related transactions.

### Card credit/debit

- Credit: moves funds from USD account → card and records transactions.
- Debit: moves funds from card → USD account and records transactions.

## 4.5 Transaction retrieval

- Endpoints:
  - `GET /transactions?page=n`
  - `GET /transactions/c/{cardId}?page=n`
  - `GET /transactions/a/{accountId}?page=n`
- Uses pageable repository queries (page size 10, sorted by creation time ascending).

---

## 5) Data Model & Persistence Strategy

The backend models a banking ledger-like domain with these key entities:

- **User**: identity + security principal details; linked to accounts, transactions, and card.
- **Account**: currency wallet per user with balance and account number.
- **Card**: single card linked to a user and card-ledger transactions.
- **Transaction**: polymorphic-like record tied to account/card context with type, amount, fee, and status.

Repositories expose derived queries such as:

- by owner UID
- by account/card IDs
- by account number
- by currency code + owner UID

This keeps business rules at service/helper level while repositories stay thin.

---

## 6) Exchange Rate Integration Workflow

`ExchangeRateService` uses `RestTemplate` to fetch latest rates from a third-party currency API.

Runtime pattern:

1. scheduler triggers initial fetch at startup.
2. scheduler repeats every 12 hours.
3. rates map is updated in memory for supported currency set.
4. conversion operations reuse cached in-memory rates instead of calling external API per request.

This reduces latency for user requests but introduces eventual consistency tied to refresh frequency.

---

## 7) Frontend Technical Workflow

## 7.1 App bootstrap and routing

- `App.js` configures routes using `createBrowserRouter`.
- Protected area (`/dashboard/*`) is wrapped with `ProtectedRoute`.
- `ProtectedRoute` checks `sessionStorage` for user presence and redirects unauthenticated users to `/login`.

## 7.2 State management model

Redux Toolkit store combines slices:

- `usersSlice`: register/login + auth status.
- `accountSlice`: account fetch/create, account search, transfer, conversion, rates.
- `cardSlice`: card fetch/create/credit/debit.
- `transactionsSlice`: paginated transaction list retrieval.
- `pageSlice`: UI-only state (spinner/navbar).

Asynchronous thunks call backend endpoints through a shared axios wrapper.

## 7.3 Auth token lifecycle in UI

1. Login thunk posts credentials to `/user/auth`.
2. Reads `Authorization` response header.
3. Stores bearer token in `sessionStorage` (`access_token`) and user object.
4. Future thunks attach token via `Authorization` header.

## 7.4 Dashboard data hydration

When `Dashboard` mounts, it dispatches:

- `fetchAccounts()`
- `fetchTransactions(0)`
- `fetchCard()`

This preloads major financial widgets and transaction table context before users perform actions.

## 7.5 User action → API workflow examples

### Transfer UI flow

- user enters recipient + currency + amount in dashboard payment flow.
- thunk posts to `/accounts/transfer` with token.
- backend returns transaction object.
- slice updates state and UI reflects success/failure status.

### Currency conversion UI flow

- user selects source/target currencies and amount.
- thunk calls `/accounts/convert`.
- resulting transaction is added and account balances are refreshed/displayed.

### Card funding/withdrawal flow

- thunks call `/card/credit` or `/card/debit`.
- state updates card balance and transaction timeline accordingly.

---

## 8) End-to-End Request Sequence (Representative)

**Example: authenticated transfer**

1. User signs in from Login page.
2. Frontend stores JWT and user in session storage.
3. User submits transfer form.
4. Redux thunk sends POST `/accounts/transfer` with bearer token.
5. Spring security filter validates token and attaches principal.
6. Account service/helper validates ownership, balance, and fee.
7. Account balances are updated in DB transaction.
8. Transaction rows are persisted.
9. API returns transfer transaction.
10. Redux state updates and UI renders latest status/ledger.

---

## 9) Design Choices Observed

## Strengths

- Clean layered backend organization (controller/service/helper/repository).
- Stateless JWT security integrated with Spring Security filter chain.
- Consistent transaction recording across account and card operations.
- Periodic FX rate caching to avoid per-request external dependency calls.
- Frontend uses predictable Redux async workflow for side effects and UI state.

## Improvement opportunities

- Financial calculations use floating-point (`double`) instead of `BigDecimal`.
- Some validation and exception handling can be standardized via global exception handlers.
- CORS and secret management are permissive/plain for production scenarios.
- Frontend auth/session handling can be hardened (refresh, expiry handling, centralized interceptors).

---

## 10) Practical Technical Workflow Summary

From an implementation perspective, the project follows this core loop:

1. **Authenticate user** (JWT issuance).
2. **Authorize each request** (JWT filter + principal context).
3. **Apply banking rule set** (service/helper validation + balance logic).
4. **Persist state changes** (JPA repositories + transaction records).
5. **Project updated state to UI** (Redux thunks/slices + React pages).
6. **Support conversion features** via scheduled exchange-rate cache refresh.

That loop is repeated consistently across accounts, cards, payments, and transaction history, which makes the codebase relatively straightforward to reason about and extend.
