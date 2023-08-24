# Determining Customer Churn and Lifetime Value
*Leveraging the BG/NBD Model and Gamma Gamma submodel to evaluate and predict customer churn, retention, and lifetime value.*

## Overview
### Purpose
Understanding customer churn and the lifetime value of your consumers are both critical questions in any business, and are especially relevant for effective, tactical marketing strategies. 

Although common analytic approaches such as clustering or classification can be effective in segmenting or predicting churn respectively, one key issue that often exists is censored data - that is, the customer's purchasing lifetime is often unfinished thus making the application of this classifciation (whether a customer has churned or not) very problematic. This is particularily relevant in non-contractual business settings where a consumer can decide to "cut ties" at any moment, so any distinguishing feature of churn needed in order to train a model would essentially be arbitrary (i.e a customer that just made a purchase yesterday may churn tomorrow, and a customer who hasn't made a purchase in 2 years may purchase again tomorrow). 

For the current analysis, we leverage "Buy 'Till You Die' probabilistic models to help account for the problem stated above, where the probability that a customer will churn (and the inverse probability of a customer's retention) are ouputted, and used to calculate a customer's lifetime value. Here, we have a more colourful and informative view on customer loyalty and consequently value, providing extremely useful information on segmenting and targeting consumers of interest.

### BG/NBD
The BTYD model of choice in our analysis is the popularly-used Beta Geometric/Negative Binomial Distribution model - a revised and improved version of the Pareto/NBD model developed in 1987.

Essentially, the model looks to predict future transactions of each individual customer by taking into account their individual purchasing history in reference to the general customer pattern. It treats purchasing phenomena as a 2 step process: 
1. Transaction Process (Buying)
As long as customer hasn't churned (is "alive), assumptions are:
  - The number of transactions made by a single consumer in a period of time follows a Poisson distribution with a transaction rate λ
  - Purchases are made randomly around their own transaction rate
  - Transaction rate varies from consumer to consumer, and follows a Gamma distribution with shape r and scale a for the entire dataset

2. Dropout Process ("Dieing"):
   - Every customer has a dropout probability p
   - Dropouts vary from consumer to consumer, and follows a Beta distribution with parameters a and β for the entire dataset


leaving the binary classification of churned/not churned 



predicting one key issues with churn 
