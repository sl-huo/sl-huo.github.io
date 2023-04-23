# Online Payment Fraud Detection

## Project Background

Applying machine learning models on a the e-commerce transactions dataset, which contains a wide range of features from device type to product features, to detect fraudulent transactions and improve the efficacy of alerts to reduce fraud loss as well as save the hassle of false positives.

![transaction](https://github.com/sl-huo/payment-fraud-detection/blob/main/asset/payment.jpeg?raw=true)
*Photo by <a href="https://unsplash.com/pt/@servuspaul?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Paul Felberbauer</a> on <a href="https://unsplash.com/photos/idNOBU5k_80?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>*

## Data Source & Description

The data comes from real-world e-commerce transactions, source: [IEEE-CIS Fraud Detection](https://www.kaggle.com/competitions/ieee-fraud-detection/data)
- Train Set: **590,540** transactions with **433** features
- Test Set: **506,691** transactions with **433** features
- Data Description:
  - `TransactionDT`: timedelta from a given reference datetime (not an actual timestamp)
  - `TransactionAMT`: transaction payment amount in USD
  - `ProductCD`: product code, the product for each transaction
  - `card1` - `card6`: payment card information, such as card type, card category, issue bank, country, etc.
  - `addr`: address
  - `dist`: distance
  - P_ and (R__) emaildomain: purchaser and recipient email domain
  - `C1`-`C14`: counting, such as how many addresses are found to be associated with the payment card, etc. The actual meaning is masked.
  - `D1`-`D15`: timedelta, such as days between previous transaction, etc.
  - `M1`-`M9`: match, such as names on card and address, etc.
  - identity information – network connection information (IP, ISP, Proxy, etc) and digital signature (UA/browser/os/version, etc) associated with transactions.

## PART 1 - Data Wrangling & EDA

### Overall Target Variable(IsFraud) & Transaction Amount Distribution

- In the training dataset, there is ~3.5% fraud cases, which is a highly imbalanced datase
- Majority of the payment amount is less than 200, while even higher proportion of fruad transactions are within and around 100

Percentage of Fraudulent Transactions   |  Transaction Amount Distribution across Two Classes
:-------------------------:|:-------------------------:
![1](https://github.com/sl-huo/payment-fraud-detection/blob/main/asset/target_distribution.png?raw=true)  |  ![2](https://github.com/sl-huo/payment-fraud-detection/blob/main/asset/transactionamt.png?raw=true)


### Card Types 

- Majority payments are through **Visa** and **MasterCard**, accounts for **65%** and **32%** respectively, with roughly ~**3.5%** of fraudulent transactions, while **discover** has the highest percentage of fraud cases, which is close to 8%
- Over 74.5% of transactions are paid via **debit card**, while 25.2% are done with **credit card** which contains highest percentage(close to 7%) of fraudulent transactions

|Transaction Distribution across Card Types|
|:-------------------------:|
|![3](https://github.com/sl-huo/payment-fraud-detection/blob/main/asset/cardtype.png?raw=true)  |


### Email Domains

- Top purchaser email domains including gmail (46%), yahoo(20%), hotmail(9%) of the known transactions
- Among the top email domains, '**mail.com**' with highest percentage of fraud cases ~20%, while '**outlook.com**' contains ~10%

|Transaction Distribution across Email Domains|
|:-------------------------:|
|![4](https://github.com/sl-huo/payment-fraud-detection/blob/main/asset/email.png?raw=true)|


### Counting Information
(such as how many addresses are found to be associated with the payment card)

Since the data for couting information is heavily right skewed, thus looking further into the higher quantile values, and it's interesting to find that for most of the columns, fraud class has much higher values and only columns `C4` & `C9` are opposite.

|Quantiles for Counting Variables|
|:-------------------------:|
|![5](https://github.com/sl-huo/payment-fraud-detection/blob/main/asset/countingcolumns.png?raw=true)|


### Web - Browser Types
- **Opera**'s percentage of fraudulent transactions is the highest among various categories with over 30%, while **Chrome** is the most popular web browser.

<!-- |Transaction Distribution across Browser Types|
|:-------------------------:| -->
![5](https://github.com/sl-huo/payment-fraud-detection/blob/main/asset/browser.png?raw=true)


## PART 2 - Analysis & Modeling

### Model Performance Comparison
- Applied **XGBoost** and **LGBM** classification models to detect fraudulent online transactions. After initial rounds of hyperparameter tuning, LGBM(AUC score: 0.94) outperformed a bit compared with XGBoost model(AUC score: 0.93).
- Runtime of LGBM is much faster compared to XGBoost (runtime: 8 minutes vs 14 seconds), which makes LGBM a preferable model for hyperparameter tuning and model optimization.

<p align="center">
  <img src="https://github.com/sl-huo/payment-fraud-detection/blob/main/asset/roc_curve.png?raw=true" />
</p>


### Feature Importance Comparison
- Look further into feature importance from two models. Overall, number of splits is much lower from LGBM while the total gain of certain features is much higher, which also indicates LGBM split the decision tree more efficiently to reach relatively same result. 
- There are quite a few feature overlaps between two models, top features with high total gain as well as high number of splits, such as `card6`(card type), `card2`(card number), `P_emaildomain`(purchaser email address), `C5` & `C7`(counting informaiton) are both identified from both models.
- Impactful features, such as `V264`, which is an engineered feature ranks the top in both models, however, the number of splts is not among the top, indicates the significant impact of this feature on the target. It would be worth further looking at the split nodes of this feature.


Feature Importance (XGBoost)   |  Feature Importance (LGBM) 
:-------------------------:|:-------------------------:
![1](https://github.com/sl-huo/payment-fraud-detection/blob/main/asset/xgb_fscore_gain.png?raw=true)  |  ![3](https://github.com/sl-huo/payment-fraud-detection/blob/main/asset/lgbm_fscore_gain.png?raw=true)
![2](https://github.com/sl-huo/payment-fraud-detection/blob/main/asset/xgb_fscore_weight.png?raw=true)  |  ![4](https://github.com/sl-huo/payment-fraud-detection/blob/main/asset/lgbm_fscore_split.png?raw=true)