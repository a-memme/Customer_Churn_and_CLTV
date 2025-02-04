# Determining Customer Churn and Lifetime Value
*Leveraging the BG/NBD Model and Gamma Gamma submodel to evaluate and predict customer churn, retention, and lifetime value.*

## Overview
### Purpose
Churn, retention and lifetime value are hot-topic terms quite commonly thrown around in the professional world  - amidst the chatter, what's certain is being able to accurately identify, calculate and understand these metrics are critical to any business looking to understand their consumer base and maximize their potential.

Although common analytic approaches such as clustering or classification can be effective in segmenting or predicting churn respectively, one key issue that often exists is the censored data present - that is, the customer's purchasing lifetime is typically unfinished, making the application of this classification (whether a customer has churned or not) very problematic. This is particularly relevant in non-contractual/non-subscription based business settings where a consumer can decide to repurchase or drop off at any moment, so any distinguishing feature of churn needed in order to train a model would essentially be arbitrary (i.e a customer that just made a purchase yesterday may churn tomorrow, and a customer who hasn't made a purchase in 2 years may purchase again tomorrow).

For the current analysis, we leverage "Buy 'Till You Die' probabilistic models to help account for the problem stated above, where in addition to predicting the number of purchases that an individual will make in a future period of time, the probability that said customer will churn is also outputted based on their historical buying behaviour. Given these outputs, we are able to calculate a customer's lifetime value over a variable period of time and begin to analyze overall retention and loyalty of our consumer base. 

The above approach allows us to make rather accurate predictions with minimal buying behaviour data, and provides a more realistic framework in expressing churn by probabilistic means rather than categorical (yes/no boolean). Given this, we are left with a set of rich information that is very easily interpretable for segmenting, targeting and actioning around our consumer base.

### Purchasing Behaviour & Churn - BG/NBD Model
The BTYD model of choice in our analysis is the popularly-used Beta Geometric/Negative Binomial Distribution model - a revised and improved version of the Pareto/NBD model developed in 1987 [^1].

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
The Gamma Gamma submodel - very popularly paired with the BG/NBD model - looks to predict the expected average profit of the consumer (i.e their future average order value) with use of the BG/NBD's outputs by a very similar process [^2]. The assumptions of the model are:
- Order frequency and monetary values (current aov) of customers are not correlated
- Monetary values of individual transactions are randomly distributed around their average order value
- Average order values vary from consumer to consumer but are constant for a single consumer through their lifetime
- Average order values follow a Gamma distribution for the entire dataset

## Method
- Preprocess data to represent inputs of the BTYD model:
  - **Recency:** time from first purchase to last purchase (in the chosen time unit - for the current analysis we choose weeks)
  - **Tenure:** time from customer's first purchase to the current date
  - **Frequency:** number of time periods a customer made a REPEAT purchase within a given timeframe (i.e the first time a customer makes a purchase = 0).
      - For ex. if a customer makes 3 seperate purchases in the same week, frequency = 1. If a customer makes 3 purchases in 3 seperate weeks, frequency = 3.
      - For the current analysis, the total time frame is ~ 1.5 years, with a chosen time interval of weeks (i.e ~ 74 weeks)
          - Time interval is chosen at the discretion of the analyst, where contextual knowledge of the data is particularly relevant
  - **Monetary Value (for CLTV):** the average order value of a consumer, dictated by the chosen time interval (weekly in the following analysis)
- Fit and validate the BG/NBD Model for predicted frequency
- Fit and assess the Gamma Gamma Model for expected avg order value
- Calculate and assess metrics of interest
    - Retention and Churn probabilities
    - CLTV
- Analyze customer churn via segmentation and sorting
- Discussion

## Results 
### Model Fitting & Validation 
*See cells 9-35 in [CLTV_and_Churn.ipynb](https://github.com/a-memme/Customer_Churn_and_CLTV/blob/main/CLTV_and_Churn.ipynb) for code*

The lifetimes module in python offers several built in functions to assess the performance/fit of the model. See their documentation here: [Lifetimes Documentation](https://lifetimes.readthedocs.io/en/latest/Quickstart.html)

#### BG/NBD Model 
- **Data vs Simulation:**

![image](https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/c871c777-6d4d-4a19-8cf9-b0ffed6ab506) 

    - Artificial dataset is generated with the fitted model's parameters
    - 0 column can be ignored as it is automatically included in generating the simulated dataset but is not included when fitting the model
  
- **Train & Test via Calibration & Holdout:**

<img width="491" alt="image" src="https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/c33bbeef-b90b-4638-b249-3577a8411420">

    - Replicating train/test process via calibration (train) and holdout (test) data splitting
        - Given a dataset of only 1 year, we only perform this process once, however future iterations would look to create a cross-validating instance 
    - Model generally overpredicts for fewer instances of repurchasing behaviour and converges as frequency values increase (up to ~14).
        - As shown below, frequencies > 14 see significantly fewer instances (<= 7 instances) which would explain why the holdout data may drastically diverge at this point. 
        - Given more data at these intervals, we'd expect averages to converge toward the general pattern.
    - MAE and RMSE values still suggest that model's performace is acceptable, with +/- predictions < 1 on average (MAE = 0.76).

<img width="873" alt="image" src="https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/c08afff7-002a-46de-a369-25b5ee9b3607">

- **Calibration/Holdout - Time Since Last Purchase & Consumer Age**

<img width="476" alt="image" src="https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/7217de84-a914-4597-8d29-92e4e71e19c5">
<img width="497" alt="image" src="https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/bccbf06e-48dd-4dcb-82ea-50c109490c5d">

    - General positive bias in the model's predictions as reflected previously, however at the least, still captures very important trends in the data, namely that:
        - As the time from the customer's last purchase increases, their average frequency of purchasing decreases
            - Areas to note may be the 10 week or 20 week periods (can leverage the "elbow-method" approach to generally choose areas of importance)
        - Consumers that are active for longer periods of time tend to see higher avg purchasing frequencies.
            - Areas to note may be ~ 35 - 45 week marks 

#### Gamma Gamma Submodel 
Unfortunately there isn't as easy of a method to train/validate the Gamma Gamma model's performance, however, we can compare its total average predictions and distribution to our dataset's for some level of confidence in performance:

<img width="1187" alt="image" src="https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/3ac85fb9-b614-49ea-9f93-3df7c4d061d7">

    - Model captures the general distribution fairly well 
    - Good MAE/RMSE values considering the majority of monetary values fall within range of $100 to ~$150 (mode = 90).

### Churn, Retention and CLTV Summary
#### Summary

<img width="1116" alt="image" src="https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/b3548e9d-465f-4c71-b35d-a3ffde3e57f3">

#### Distributions
<img width="1142" alt="image" src="https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/7c114eea-f8c1-4c63-b941-9d1b921df7b5">
<img width="1135" alt="image" src="https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/9bdba7f2-433d-422d-a36a-589097b880cb">

    - Notable distinctions in the distributions of churn and cltv values 
        - Highest frequency of consumers on polarizing ends of churn probabilities, indicating a potentially large base of customers to action around.
        - Right-skewed distribution in cltv predictions despite the high number of high-rentention/low churn probability customers - explore this relationship further in the Analysis section below.

### Probability "Alive" - Analyzing Individual Retention Life 
#### Highest Retention:
<img width="468" alt="image" src="https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/fff4d10d-2e0f-43ce-a8e1-1d98623d8617">

- In the above visual we can see the retention history for the consumer with the best retention probability calculated by the model.
- As witnessed, the retention probability begins to decrease at an exponential rate the longer the consumer goes without purchasing
- Once the consumer makes a purchase, their retention probability increases, and then continues to decrease once again, but at a slower rate.
  
#### Comparing Retention Histories

##### Figure A
<img width="492" alt="image" src="https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/11c03fff-7c80-4697-8ff2-4df68d7f6765">

##### Figure B
<img width="467" alt="image" src="https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/c593c617-8341-454e-89db-fcdab8ab9cdd">


- In the above visual, Figure A represents the consumer with the worst current retention probability, seeing a similar phenomena where the consumer's retention probability decreases at an exponential rate as time goes by without repeat purchase.
- Figure B represents another consumer with the same recency and tenure as Figure A, however, with a much better retention probability despite having a lower frequency of purchase - Why?
    - For the model's calculation, purchasing pattern is also very important in determining probability of retention - i.e although Figure A had a much larger frequency v Figure B within the same period T, they deviated quite substantially from their typical purchasing pattern in comparison, and thus, see retention probability drop at a faster pace.
- Figure A is a perfect example of a missed opportunity - here we had a frequent and consistent repeat consumer with great retention through a large area of their purchasing life, only to see that retention drop off drastically. At any moment from >= week 45, there were several opportunities to take action with aim to prevent this drop off given the information at hand.
 
## Analysis
*See cells 37-43 in [CLTV_and_Churn.ipynb](https://github.com/a-memme/Customer_Churn_and_CLTV/blob/main/CLTV_and_Churn.ipynb) for code*

Through validation and exploration of our model, we can now perform some simple segmenting/sorting tasks to identify consumers of various value characteristics.

### Retention X CLTV Segmentation
- Segmenting retention and cltv customers by quadrant (A = highest; D = lowest)

![image](https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/dd41b397-3f0c-4cf9-a540-dbf84d8b53b9)

    - Although high retention/high cltv and low retention/low cltv buckets make up the largest proportions, we can clearly see that high retention doesn't always lead to the highest cltv, and vice versa

### High Valued Customers
- **Best Retention / Churn**
  
<img width="692" alt="image" src="https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/67d332c0-84a5-456b-9d02-3ef0721d3a70">

    - In line with our representation of churn history, consumers with best retention/churning probabilities all have long histories of purchase (recency), have purchased recently (T - recency --> usually within the past 0-1 weeks) and have purchased frequently through that life (frequency).
        - This gives us further confirmation in the model's outputs regarding retention
    - Consumers in this distinction may be appropriate candidates for incentivization via rewards and/or strategical upselling given their high likihood of repurchase and varying aov/predicted lifetime values.

- **Most Valuable**
<img width="1152" alt="image" src="https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/2cf2ede0-d430-4f04-8f39-b48ff74aa26d">

    - When we segment customers by churn probabilities, we can filter for the top quadrant and understand who are considered the most valuable based on our yearly prediction of cltv.
    - Consumers in this distinction could be particularly appropriate candidates for rewards targeting and other marketing techniques revolved around nurturing customer relationships.
 
### High Probability "Churners"
In contrast, we most likely will look to identify consumers who have a higher liklihood of churning, but possess some characteristic leading us to believe that some form of incentivization or target marketing may elicit repurchase once again. 

- One way in which we can identify said consumers is use our churning distribution knowledge and filter for the most frequently occuring high churn probability groups (p>=0.8), and sort based on predicted lifetime value (most valuable to least):
    -  Here, we can identify consumers in the lower 2 churning segments and look to understand who is the most valuable, specifics on their overall purchasing behaviour, and map out the appropriate amount of actionable effort against the expected gain or profit in "re-igniting" their purchasing behaviour.
    -  Futhermore, said customers could be good candidates for feedback surveying.

<img width="1190" alt="image" src="https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/11260e3c-8ae0-4b75-9bff-d0d921c7e265">

- More simply, we can also filter for a lower churning segment (C in this case) and compare and contrast cltv segments to simply visualize who to target (for ex. we may want to specifically look at targeting C level churners that have an A level predicted lifetime value in this case):

  <img width="1194" alt="image" src="https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/d9e2f57e-fb05-4ab5-a058-ca414e7b2b9c">

- Finally, we could look to be more proactive by identifying consumers at an earlier stage of the process who are in more of a "potential risk" category, and are more appropriate candidates for simple probing/reminder styles of targeting to help stimulate repurchase at a time where we typically see frequency and/or retention drop off.
    - One way we could assess this group is to leverage our understanding of customer age and/or time since last purchase to filter through our data. Recall that between 5 to 15 weeks since last purchase is where we saw the largest drop off in average purchasing frequency. Below, we use this information to filter through our data and sort by churn probability:

<img width="1197" alt="image" src="https://github.com/a-memme/Customer_Churn_and_CLTV/assets/79600550/dccc5077-d292-4bae-8031-e4d16728ec4b">

## Conclusion
In the above analysis, we leverage Buy Till You Die probabilistic models to calculate customer retention and churn, and ultimately, predict ltv at a duration of our choosing. By expressing churn in probabilistic means, we are able to represent the dynamic nature of retention and lifetime value more appropriately, accounting for the innate censored-nature of the data at hand. Furthermore, we can benefit from some typical merits of the BG/NBD model, namely speedy optimization, easy deployment and rich interpretability as exemplified above. Finally, the rich output from the model allows us to sort, segment and analyze our consumers with a variety of approaches.

### Limitations 
- **Long Purchasing History**
    - Although the size of our dataset is large enough for model creation, ideally we would like to see a longer time frame where a greater number of observations per consumer can be witnessed. As seen in the model above, as frequencies per consumer increase, we see the model converge and become more accurate (freq>= 6). In addition, a longer timeframe would allow for a more robust cross validation process.
- **Flexibility**
    - Although a potential benefit of BG/NBD is its input simplicity, businesses may WANT to incorporate consumer demographic data into the model to understand profiles of customers likely to have greater retention. This type of flexibility may also be useful in understanding first-time customers
        - Alternatively, one could look to  others' instances of incorporating non-transactional data into BG/NBD models for inspiration [^3].
- **Repeat Customer Analysis ONLY**
    - Our analysis only looks at REPEAT customers and ignores those that have only made single purchases, which make up an extremely large majority of the total customer base (in our case 74% of our entire consumer base -- *see cells 6-7 in  [CLTV_and_Churn.ipynb](https://github.com/a-memme/Customer_Churn_and_CLTV/blob/main/CLTV_and_Churn.ipynb)* )

### Future Outlook
Future analyses may look to:
- Incorporate the power of combining other models to greater understand our consumer base - for example, mapping demographic/taxonomical data to our model's outputs and performing correlation analysis.
- Engage in model maintenance and tuning when more historic purchasing history is available.
- Engage in more sophisticated segmentation techniques, such as kmeans clustering to identify and analyze segments with greater accuracy based on grouping distance rather than arbitrary cut-offs.
- Explore alternative methods or combined model approaches in understanding single-transaction consumer behaviour, retention probability, and lifetime value, and explore how said relationships may differ vs repeat customer behaviour.

## References
[^1]: Fader, Peter & Hardie, Bruce & Lee, Ka. (2005). “Counting Your Customers” the Easy Way: An Alternative to the Pareto/NBD Model. Marketing Science. 24. 275-284. 10.1287/mksc.1040.0098. Available at https://www.brucehardie.com/papers/bgnbd_2004-04-20.pdf
[^2]: Hardie, B.G. The Gamma-Gamma Model of Monetary Value. Available at https://www.brucehardie.com/notes/025/gamma_gamma.pdf 
[^3]: Van Oest, Rutger and Knox, George, Extending the BG/NBD: A Simple Model of Purchases and Complaints (November 16, 2010). International Journal of Research in Marketing, Forthcoming, Available at SSRN: https://ssrn.com/abstract=1711186
