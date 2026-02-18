
# Credit Card Fraud Detection using Machine Learning and Deep Learning

## 👥 Team

* Maanasa Prathap Chander \[2022503065]
* Gopika S \[2022503505]
* Neeraja A \[2022503581]
* Venkataswathi M \[2022503701]

## 🧠 Project Overview

This project aims to build an intelligent fraud detection system capable of identifying fraudulent credit card transactions with high accuracy. Given the extreme class imbalance (fraud accounts for less than 0.2% of all records), this work explores both traditional and deep learning techniques, complemented by various data balancing strategies.

## 📁 Dataset

* **Source:** Credit Card Fraud Detection Dataset (Anonymized)
* **Size:** 284,807 transactions
* **Fraudulent transactions:** 492 (\~0.172%)
* **Features:** 30 features (Time, V1–V28 PCA features, Amount, Class)

## 🎯 Objectives

* Detect fraudulent transactions using machine learning and deep learning models.
* Address class imbalance with techniques such as under-sampling, over-sampling, and **SMOTE**.
* Compare models: Decision Tree, Random Forest, and Fully Connected Neural Networks.
* Minimize false positives to reduce unnecessary manual review.
* Demonstrate the adaptability of these models for other anomaly detection problems.

## 🏗️ Methodology

### 💡 Models Used

* Decision Tree Classifier
* Random Forest Classifier
* Fully Connected Neural Network (implemented in TensorFlow/Keras)

### ⚖️ Data Balancing Techniques

* **Under-sampling**: Reducing majority class samples.
* **Over-sampling**: Replicating minority class samples.
* **SMOTE**: Synthesizing new minority class samples.

### 🧪 Experiments

* Training and evaluating models using original and balanced datasets.
* ROC-AUC, confusion matrix, and F1-score used for performance evaluation.

## 🔍 Key Results

The best results are achieved by over-sampling the under-represented class using SMOTE (synthetic minority oversampling technique). With this approach, the model is able to detect 100% of all fraudulent transactions in the unseen test set. This fully satisfies the primary objective to detect the vast majority of abnormal transactions. Please note that the technique and model used are simple to implement simple, easy to use and can be updated in real-time.

In addition, the number of false positive remains acceptable. This means a lot less verification work (on legitimate transactions) for the fraud departement compare dto some other approaches which failed on this aspect. Key results are shown below:

## 📊 Visualizations
![Confusion matrix achieved using SMOTE over-sampling and a simple dense neural network](https://github.com/maanasaprathap/credit-card-fraud-detection/blob/main/images/main%20cm.png)
![Comparison of key performance indicators between the tested approaches](https://github.com/maanasaprathap/credit-card-fraud-detection/blob/main/images/main%202.png)
## 📂 Repository Structure

```
├── Deep Learning + Sampling.ipynb  # Jupyter Notebook with full model development
├── Data mining project.pdf         # Project report (Anna University)
├── README.md                       # Project overview and instructions
```

## 🚀 Getting Started

### Prerequisites

* Python 3.8+
* Jupyter Notebook
* Install required libraries:

```bash
pip install pandas numpy scikit-learn imbalanced-learn matplotlib seaborn tensorflow
```

### Running the Notebook

1. Launch Jupyter:

   ```bash
   jupyter notebook
   ```
2. Open `Deep Learning + Sampling.ipynb`.
3. Run all cells in sequence.

## 🏁 Conclusion

This project demonstrates that combining deep learning with effective sampling techniques like SMOTE significantly enhances fraud detection in highly imbalanced datasets. The methodology is generalizable and applicable to various anomaly detection scenarios.

