# Determining Customer Churn and Lifetime Value
*Leveraging the BG/NBD Model and Gamma Gamma submodel to evaluate and predict customer churn, retention, and lifetime value.*

## Overview
### Purpose
Churn, retention and lifetime value are all trigger terms thrown around constantly in the professional world  - being able to accurately identify, calculate and understand these metrics are critical to any business looking to understand their consumer base and maximize their potential.

Although common analytic approaches such as clustering or classification can be effective in segmenting or predicting churn respectively, one key issue that often exists is the censored data present - that is, the customer's purchasing lifetime is typically unfinished, making the application of this classification (whether a customer has churned or not) very problematic. This is particularly relevant in non-contractual business settings where a consumer can decide to "cut ties" at any moment, so any distinguishing feature of churn needed in order to train a model would essentially be arbitrary (i.e a customer that just made a purchase yesterday may churn tomorrow, and a customer who hasn't made a purchase in 2 years may purchase again tomorrow).

For the current analysis, we leverage "Buy 'Till You Die' probabilistic models to help account for the problem stated above, where in addition to predicting the number of purchases that an individual will make in a future period of time, the probability that said customer will churn is also outputted based on their historical buying behaviour. Given these outputs, we are able to calculate a customer's lifetime value over a variable period of time and begin to analyze overall retention and loyalty of our consumer base. The above approach allows us to make rather accurate predictions with minimal buying behaviour data, and provides a more realistic framework in expressing churn by probabilistic means rather than categorical (yes/no boolean). Given this, we are left with a set of rich information that is very easily interpretable for segmenting, targeting and actioning around our consumer base.

### Purchasing Behaviour & Churn - BG/NBD Model
The BTYD model of choice in our analysis is the popularly-used Beta Geometric/Negative Binomial Distribution model - a revised and improved version of the Pareto/NBD model developed in 1987.

Essentially, the model looks to predict future transactions of each individual customer by taking into account their individual purchasing history in reference to the general customer pattern. It treats purchasing phenomena as a two step process: 

**1. Transaction Process ("Buying")**

As long as customer hasn't churned (is "alive), assumptions of the model are:
  - The number of transactions made by a single consumer in a period of time follows a Poisson distribution with a transaction rate λ.
  - Purchases for individual consumers are randomly distributed around their own transaction rate λ.
  - Transaction rate λ varies from consumer to consumer, but follows a Gamma distribution with shape r and scale a for the entire dataset

**2. Dropout Process ("Dieing"):**
   - Every customer has a dropout probability p
   - Dropouts vary from consumer to consumer, and follows a Beta distribution with parameters a and β for the entire dataset

### CLTV - Gamma Gamma Submodel
The Gamma Gamma submodel - very popularly paired with the BG/NBD model - looks to predict the expected average profit of the consumer (i.e their future average order value) with use of the BG/NBD's outputs by a very similar process. The assumptions of the model are:
- Order frequency and monetary values (current aov) of customers are not correlated
- Monetary values of individual transactions are randomly distributed around their average order value
- Average order values vary from consumer to consumer but are constant for a single consumer through their lifetime
- Average order values follow a Gamma distribution for the entire dataset

  ## Method
- Preprocess data to represent inputs of the BTYD model:
  - Recency: time from first purchase to last purchase (in the chosen time unit - for the current analysis we choose weeks)
  - Tenure: time from customer's first purchase to the current date
  - Frequency: number of time periods a customer made a REPEAT purchase within a given timeframe (i.e the first time a customer makes a purchase isn't included where freq = 0).
      - For ex, if a customer makes 3 seperate purchases in the same week, frequency is 1. If a customer makes 3 purchases in 3 seperate weeks, frequency is 3.
      - For the current analysis, the total time frame is ~ 1.5 years, with a chosen time period of weeks (i.e ~74 weeks)
          - Time period is chosen at the discretion of the analyst, where contextual knowledge of the data is particularly relevant
      - Frequencies of 0 mean that a customer has made a purchase, but hasn't had a REPEAT transaction.
  - Monetary Value (for CLTV): the average order value of a consumer, dictated by the chosen time interval (i.e avg **weekly** order value in our case)
- Fit and validate the BG/NBD Model for predicted frequency
- Fit and assess the Gamma Gamma Model for expected avg order value
- Calculate and assess metrics of interest
    - Retention and Churn probabilities
    - CLTV
- Analyze customer churn via segmentation and sorting
- Discussion

## Results 
### Model Fitting & Validation 
The lifetimes module in python offers several built in functions to assess the performance/fit of the model:

#### BG/NBD Model 
- **Data vs Simulation:**

![image](https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/c871c777-6d4d-4a19-8cf9-b0ffed6ab506) 


    - Artificial dataset is generated with the fitted model's parameters
    - 0 column can be ignored as it is automatically included in generating the simulated dataset but is not included in our set when fitting the model
        - The modified version of this model (MBG/NBD model) is meant to account for non-repeat purchasers, however, given the innate limited information on their buying history, the retention probability outputted by the model is generally unreliable regardless.
  
- **Train & Test via Calibration & Holdout:**

<img width="473" alt="image" src="https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/c36be948-0712-4d47-a387-b2605183d93e">

    - Replicating train/test process via calibration (train) and holdout (test) data splitting
        - Given a dataset of only 1 year, we only perform this process once, however future iterations would look to create a cross-validating instance via repetition
    - Model generally overpredicts for fewer instances of re-purchasing behaviour and converges as the numbers increase (up to ~16).
        - Freq > ~16 we see the holdout data divert more instensly away from calibration, however, this is likely due to limited instances of purchasing behaviour > 16.
    - MAE and RMSE values still suggest that model's performace is still acceptable, over/under predicting frequency behaviour by a value of < 1 on average (MAE = 0.76).

- **Calibration/Holdout - Time Since Last Purchase & Consumer Age**

<img width="476" alt="image" src="https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/7217de84-a914-4597-8d29-92e4e71e19c5">
<img width="497" alt="image" src="https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/bccbf06e-48dd-4dcb-82ea-50c109490c5d">

    - As reflected in the previous visual, we see a general positive bias in the model's predictions, converging as we gain more information about the consumer's purchasing behaviour
    - Nevertheless, the model still generally captures captures very important trends effectively, namely:
        - As the time from the customer's last purchase increases, their average frequency of purchasing decreases exponentially 
            - Areas to note may be the 10 week or 20 week periods (pseudo "elbow-method" approach or when avg frequency levels off)
        - Consumers that are active for longer periods of time tend to see higher avg purchasing frequencies.
            - Areas to note may be ~35 - 45 week marks 

#### Gamma Gamma Submodel 
Unfortunately there isn't as easy of method to train/validate the Gamma Gamma model's performance, however, we can compare its total average predictions and distribution to our dataset's for at least some level of confidence in its performance:

<img width="1187" alt="image" src="https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/3ac85fb9-b614-49ea-9f93-3df7c4d061d7">

    - Model capture the general distirbution fairly well 
    - Performance metrics quite good considering the majority of values falling in range of just below $100 to ~$150 (mode = 90).

### Churn, Retention and CLTV Summary
#### Summary

<img width="1116" alt="image" src="https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/b3548e9d-465f-4c71-b35d-a3ffde3e57f3">

    - Large ranges apparent for both retention/churn probabilities and cltv values
    
#### Distributions
<img width="1142" alt="image" src="https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/7c114eea-f8c1-4c63-b941-9d1b921df7b5">
<img width="1135" alt="image" src="https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/9bdba7f2-433d-422d-a36a-589097b880cb">

    - Notable distinctions in the distrbutions of churn and computed cltv values 
        - Highest frequency of consumers on polarizing ends of churn probabilities, of course indicating a large number of customers to potentially target for incentivization. 
        - Right-skewed distirbution of cltv despite the high number of high-rentention/low churn probability customers - uncover this phenomena further in the Analysis section below.






