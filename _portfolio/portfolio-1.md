---
title: "Credit Card Fraud Detection"
excerpt: "Using XG Boost in Python<br/><img src='/images/portfolio-1-image.jpeg'>"
collection: portfolio
---

## Fraud Detection Tutorial

1. Using the credit card fraud detection dataset from Kaggle, I am going to use gradient boosted tree to identify fraudulant cases.

2. Once the model performs well, I will try and create simple Web App, where it shows the current credit card usage and show the location of the fraudulant activity was held. (This part will be done using kubernetes docker image that has Apache Spark streaming and Kafka on GCP). 


First import the usual suspects. 


```python
import pandas as pd 
import numpy as np
import sklearn
from xgboost import XGBClassifier
from sklearn.model_selection import StratifiedKFold, cross_val_score

```

Reading from the csv file "creditcard.csv"


```python
df = pd.read_csv("creditcard.csv")


```


```python
df.describe().transpose()


```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>count</th>
      <th>mean</th>
      <th>std</th>
      <th>min</th>
      <th>25%</th>
      <th>50%</th>
      <th>75%</th>
      <th>max</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Time</th>
      <td>284807.0</td>
      <td>9.481386e+04</td>
      <td>47488.145955</td>
      <td>0.000000</td>
      <td>54201.500000</td>
      <td>84692.000000</td>
      <td>139320.500000</td>
      <td>172792.000000</td>
    </tr>
    <tr>
      <th>V1</th>
      <td>284807.0</td>
      <td>1.165980e-15</td>
      <td>1.958696</td>
      <td>-56.407510</td>
      <td>-0.920373</td>
      <td>0.018109</td>
      <td>1.315642</td>
      <td>2.454930</td>
    </tr>
    <tr>
      <th>V2</th>
      <td>284807.0</td>
      <td>3.416908e-16</td>
      <td>1.651309</td>
      <td>-72.715728</td>
      <td>-0.598550</td>
      <td>0.065486</td>
      <td>0.803724</td>
      <td>22.057729</td>
    </tr>
    <tr>
      <th>V3</th>
      <td>284807.0</td>
      <td>-1.373150e-15</td>
      <td>1.516255</td>
      <td>-48.325589</td>
      <td>-0.890365</td>
      <td>0.179846</td>
      <td>1.027196</td>
      <td>9.382558</td>
    </tr>
    <tr>
      <th>V4</th>
      <td>284807.0</td>
      <td>2.086869e-15</td>
      <td>1.415869</td>
      <td>-5.683171</td>
      <td>-0.848640</td>
      <td>-0.019847</td>
      <td>0.743341</td>
      <td>16.875344</td>
    </tr>
    <tr>
      <th>V5</th>
      <td>284807.0</td>
      <td>9.604066e-16</td>
      <td>1.380247</td>
      <td>-113.743307</td>
      <td>-0.691597</td>
      <td>-0.054336</td>
      <td>0.611926</td>
      <td>34.801666</td>
    </tr>
    <tr>
      <th>V6</th>
      <td>284807.0</td>
      <td>1.490107e-15</td>
      <td>1.332271</td>
      <td>-26.160506</td>
      <td>-0.768296</td>
      <td>-0.274187</td>
      <td>0.398565</td>
      <td>73.301626</td>
    </tr>
    <tr>
      <th>V7</th>
      <td>284807.0</td>
      <td>-5.556467e-16</td>
      <td>1.237094</td>
      <td>-43.557242</td>
      <td>-0.554076</td>
      <td>0.040103</td>
      <td>0.570436</td>
      <td>120.589494</td>
    </tr>
    <tr>
      <th>V8</th>
      <td>284807.0</td>
      <td>1.177556e-16</td>
      <td>1.194353</td>
      <td>-73.216718</td>
      <td>-0.208630</td>
      <td>0.022358</td>
      <td>0.327346</td>
      <td>20.007208</td>
    </tr>
    <tr>
      <th>V9</th>
      <td>284807.0</td>
      <td>-2.406455e-15</td>
      <td>1.098632</td>
      <td>-13.434066</td>
      <td>-0.643098</td>
      <td>-0.051429</td>
      <td>0.597139</td>
      <td>15.594995</td>
    </tr>
    <tr>
      <th>V10</th>
      <td>284807.0</td>
      <td>2.239751e-15</td>
      <td>1.088850</td>
      <td>-24.588262</td>
      <td>-0.535426</td>
      <td>-0.092917</td>
      <td>0.453923</td>
      <td>23.745136</td>
    </tr>
    <tr>
      <th>V11</th>
      <td>284807.0</td>
      <td>1.673327e-15</td>
      <td>1.020713</td>
      <td>-4.797473</td>
      <td>-0.762494</td>
      <td>-0.032757</td>
      <td>0.739593</td>
      <td>12.018913</td>
    </tr>
    <tr>
      <th>V12</th>
      <td>284807.0</td>
      <td>-1.254995e-15</td>
      <td>0.999201</td>
      <td>-18.683715</td>
      <td>-0.405571</td>
      <td>0.140033</td>
      <td>0.618238</td>
      <td>7.848392</td>
    </tr>
    <tr>
      <th>V13</th>
      <td>284807.0</td>
      <td>8.176030e-16</td>
      <td>0.995274</td>
      <td>-5.791881</td>
      <td>-0.648539</td>
      <td>-0.013568</td>
      <td>0.662505</td>
      <td>7.126883</td>
    </tr>
    <tr>
      <th>V14</th>
      <td>284807.0</td>
      <td>1.206296e-15</td>
      <td>0.958596</td>
      <td>-19.214325</td>
      <td>-0.425574</td>
      <td>0.050601</td>
      <td>0.493150</td>
      <td>10.526766</td>
    </tr>
    <tr>
      <th>V15</th>
      <td>284807.0</td>
      <td>4.913003e-15</td>
      <td>0.915316</td>
      <td>-4.498945</td>
      <td>-0.582884</td>
      <td>0.048072</td>
      <td>0.648821</td>
      <td>8.877742</td>
    </tr>
    <tr>
      <th>V16</th>
      <td>284807.0</td>
      <td>1.437666e-15</td>
      <td>0.876253</td>
      <td>-14.129855</td>
      <td>-0.468037</td>
      <td>0.066413</td>
      <td>0.523296</td>
      <td>17.315112</td>
    </tr>
    <tr>
      <th>V17</th>
      <td>284807.0</td>
      <td>-3.800113e-16</td>
      <td>0.849337</td>
      <td>-25.162799</td>
      <td>-0.483748</td>
      <td>-0.065676</td>
      <td>0.399675</td>
      <td>9.253526</td>
    </tr>
    <tr>
      <th>V18</th>
      <td>284807.0</td>
      <td>9.572133e-16</td>
      <td>0.838176</td>
      <td>-9.498746</td>
      <td>-0.498850</td>
      <td>-0.003636</td>
      <td>0.500807</td>
      <td>5.041069</td>
    </tr>
    <tr>
      <th>V19</th>
      <td>284807.0</td>
      <td>1.039817e-15</td>
      <td>0.814041</td>
      <td>-7.213527</td>
      <td>-0.456299</td>
      <td>0.003735</td>
      <td>0.458949</td>
      <td>5.591971</td>
    </tr>
    <tr>
      <th>V20</th>
      <td>284807.0</td>
      <td>6.406703e-16</td>
      <td>0.770925</td>
      <td>-54.497720</td>
      <td>-0.211721</td>
      <td>-0.062481</td>
      <td>0.133041</td>
      <td>39.420904</td>
    </tr>
    <tr>
      <th>V21</th>
      <td>284807.0</td>
      <td>1.656562e-16</td>
      <td>0.734524</td>
      <td>-34.830382</td>
      <td>-0.228395</td>
      <td>-0.029450</td>
      <td>0.186377</td>
      <td>27.202839</td>
    </tr>
    <tr>
      <th>V22</th>
      <td>284807.0</td>
      <td>-3.444850e-16</td>
      <td>0.725702</td>
      <td>-10.933144</td>
      <td>-0.542350</td>
      <td>0.006782</td>
      <td>0.528554</td>
      <td>10.503090</td>
    </tr>
    <tr>
      <th>V23</th>
      <td>284807.0</td>
      <td>2.578648e-16</td>
      <td>0.624460</td>
      <td>-44.807735</td>
      <td>-0.161846</td>
      <td>-0.011193</td>
      <td>0.147642</td>
      <td>22.528412</td>
    </tr>
    <tr>
      <th>V24</th>
      <td>284807.0</td>
      <td>4.471968e-15</td>
      <td>0.605647</td>
      <td>-2.836627</td>
      <td>-0.354586</td>
      <td>0.040976</td>
      <td>0.439527</td>
      <td>4.584549</td>
    </tr>
    <tr>
      <th>V25</th>
      <td>284807.0</td>
      <td>5.340915e-16</td>
      <td>0.521278</td>
      <td>-10.295397</td>
      <td>-0.317145</td>
      <td>0.016594</td>
      <td>0.350716</td>
      <td>7.519589</td>
    </tr>
    <tr>
      <th>V26</th>
      <td>284807.0</td>
      <td>1.687098e-15</td>
      <td>0.482227</td>
      <td>-2.604551</td>
      <td>-0.326984</td>
      <td>-0.052139</td>
      <td>0.240952</td>
      <td>3.517346</td>
    </tr>
    <tr>
      <th>V27</th>
      <td>284807.0</td>
      <td>-3.666453e-16</td>
      <td>0.403632</td>
      <td>-22.565679</td>
      <td>-0.070840</td>
      <td>0.001342</td>
      <td>0.091045</td>
      <td>31.612198</td>
    </tr>
    <tr>
      <th>V28</th>
      <td>284807.0</td>
      <td>-1.220404e-16</td>
      <td>0.330083</td>
      <td>-15.430084</td>
      <td>-0.052960</td>
      <td>0.011244</td>
      <td>0.078280</td>
      <td>33.847808</td>
    </tr>
    <tr>
      <th>Amount</th>
      <td>284807.0</td>
      <td>8.834962e+01</td>
      <td>250.120109</td>
      <td>0.000000</td>
      <td>5.600000</td>
      <td>22.000000</td>
      <td>77.165000</td>
      <td>25691.160000</td>
    </tr>
    <tr>
      <th>Class</th>
      <td>284807.0</td>
      <td>1.727486e-03</td>
      <td>0.041527</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>



There are 284807 observations. 
transpose() method is one of my favorite when the data has many columns. 
"Class" must be the identifier


```python
# Normal Transactions
len(df[df["Class"] == 0])

```




    25749




```python
# Fraud Transactions
len(df[df["Class"] == 1])

```




    88



You can see that there is definite class imbalance in this dataset. 
Solution: 
1. up sample the minority class
2. down sample the majority class



```python
from sklearn.utils import resample 

# I will down sample the majority class
n_df = df[df["Class"] == 0]
f_df = df[df["Class"] == 1]

# Downsample was done randomly to match the length of the fradulant transactions. 

n_df_downsample = resample(n_df, 
                          replace = False, 
                          n_samples = len(f_df),
                          random_state=111)

len(n_df_downsample)

```




    88



Now, we combine the down sampled df with the f_df as the training set. 



```python
train_df = n_df_downsample.append(f_df)

x = train_df.values[:, 0:30].astype(float)
y = train_df.values[:, 30] # target "Class"




```

Use XG Boost and cross validate using the area under the curve for the ROC. 
- ROC curve is used for Sensitivity / Specificity report. 
- Sensitivity = True positive rate
- Specificity = False positive rate 
- The area under the roc curve = ROC AUC is the measure of how well a parameter can distinguish between two diagnostic groups in this case Normal vs Fradulant 
- upper left corner of the ROC curve is something to look for, more coverage, means better results 



```python
m = XGBClassifier()
kfold = StratifiedKFold(n_splits=10, random_state = 1)

result = cross_val_score(m, x, y, cv=kfold, scoring = "roc_auc")

print("AUC: ", result.mean())


```

    AUC:  0.9934992283950617


Looks like it is producing really high result. 
