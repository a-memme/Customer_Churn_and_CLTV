# Determining Customer Churn and Lifetime Value
*Leveraging the BG/NBD Model and Gamma Gamma submodel to evaluate and predict customer churn, retention, and lifetime value.*

## Overview
### Purpose
Churn, retention and lifetime value are all trigger terms thrown around constantly in the professional world  - being able to accurately identify, calculate and understand these metrics are critical to any business looking to understand their consumer base and maximize their potential.

Although common analytic approaches such as clustering or classification can be effective in segmenting or predicting churn respectively, one key issue that often exists is the censored data present - that is, the customer's purchasing lifetime is typically unfinished, making the application of this classification (whether a customer has churned or not) very problematic. This is particularly relevant in non-contractual business settings where a consumer can decide to "cut ties" at any moment, so any distinguishing feature of churn needed in order to train a model would essentially be arbitrary (i.e a customer that just made a purchase yesterday may churn tomorrow, and a customer who hasn't made a purchase in 2 years may purchase again tomorrow).

For the current analysis, we leverage "Buy 'Till You Die' probabilistic models to help account for the problem stated above, where in addition to predicting the number of purchases that an individual will make in a future period of time, the probability that said customer will churn is also outputted based on their historical buying behaviour. Given these outputs, we are able to calculate a customer's lifetime value over a variable period of time and begin to analyze overall retention and loyalty of our consumer base. The above approach allows us to make rather accurate predictions with minimal buying behaviour data, and provides a more realistic framework in expressing churn by 
probabilistic means rather than categorical (yes/no boolean). Given this, we are left with a set of rich information that is very easily interpretable for segmenting, targeting and actioning around our consumer base.

### Purchasing Behaviour & Churn - BG/NBD Model
The BTYD model of choice in our analysis is the popularly-used Beta Geometric/Negative Binomial Distribution model - a revised and improved version of the Pareto/NBD model developed in 1987.

Essentially, the model looks to predict future transactions of each individual customer by taking into account their individual purchasing history in reference to the general customer pattern. It treats purchasing phenomena as a two step process: 

**1. Transaction Process ("Buying")**

As long as customer hasn't churned (is "alive), assumptions of the model are:
  - The number of transactions made by a single consumer in a period of time follows a Poisson distribution with a transaction rate λ.
  - Purchases for individual consumers are randomly distributed around their own transaction rate λ.
  - Transaction rate λ varies from consumer to consumer, and follows a Gamma distribution with shape r and scale a for the entire dataset

**2. Dropout Process ("Dieing"):**
   - Every customer has a dropout probability p
   - Dropouts vary from consumer to consumer, and follows a Beta distribution with parameters a and β for the entire dataset

### CLTV - Gamma Gamma Submodel
The Gamma Gamma submodel - very popularly paired with the BG/NBD model - looks to predict the expected average profit of the consumer (i.e their future average order value) by a very similar process. The assumptions of the model are:
- Order frequency and monetary values (current aov) of customers are not correlated
- Monetary values of individual transactions are randomly distributed around their average order value
- Average order values vary from consumer to consumer but are constant for a single consumer through their lifetime
- Average order values of the entire dataset follow a Gamma distribution

  ## Method
- Preproocess data to represent inputs of the BTYD model:
  - Recency: time from first purchase to last purchase (in the chosen time unit - for the current analysis we choose weeks)
  - Tenure: time from customer's first purchase to the current date
  - Frequency: number of time periods a customer made a  purchase within a given timeframe (i.e the first time a customer makes a purchase isn't included; freq = 0).
      - For ex, if a customer makes 3 seperate purchases in the same week, frequency is 1. If a customer makes 3 purchases in 3 seperate weeks, frequency is 3.
      - For the current analysis, the total time frame is ~ 1.5 years, with a chosen time period of weeks (i.e ~74 weeks)
          - Time period is chosen at the discretion of the analyst, where contextual knowledge of the data is particularly relevant
      - Frequencies of 0 mean that a customer has made a purchase, but hasn't had a REPEAT transaction.
  - Monetary Value (for CLTV): the average order value for a customer per chosen time period (i.e avg weekly order value in our case)
  
- Fit and validate the BG/NBD Model
    - Modified BG/NBD model is used in our case to account for frequencies of 0.
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
- Comparing our current dataset with an articifically simulated dataset, fit with the same paramaters as our fitted model
    - Modified BG/NBD is fit to include frequencies of 0, which is also easier to compare to the simulated dataset that also includes 0's.
![image](https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/d1575af1-1b17-43bd-8408-9e508c37dd1c)

- 
