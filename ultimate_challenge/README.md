# Takehome Challenge 1 - Ultimate

### Part 1 - Exploratory Data Analysis

_Aggregate these login counts based on 15 ­minute time intervals, and visualize and describe the resulting time series of login counts in ways that best characterize the underlying patterns of the demand. Report any data quality issues._

The data was read in via `logins = pd.read_json('logins.json')`  
Aggregation was done with: `logins.resample('15T', on='login_time').count()`:  
![login_15.head()](https://raw.githubusercontent.com/claireramming/Springboard-Mini-Projects/master/ultimate_challenge/imgs/15aggimg.png)  
I then checked for missing data, if there are missing time blocks they should show as NaN. `login_15[login_15.login_count.isnull()]` which yielded no results.  
Next I looked at the daily login counts. January 1st, 1970 is a Thursday, so expanding that out, the peaks in logins tend to come on the weekends and are lowest on Mondays. The low first day is due to the data starting at 8pm on January 1st.  
![Daily Logins](https://raw.githubusercontent.com/claireramming/Springboard-Mini-Projects/master/ultimate_challenge/imgs/dailylogins.png)  
The day with the highest login count (4/4/1970, which is a Saturday) also takes up 7 of the 10 highest login count intervals. The top interval (3/1/1970 at 4:30am) is on a Thursday. `login_15.sort_values('login_count', ascending=False).head(10)`

	                     login_count    
    login_time                      
    1970-03-01 04:30:00           73
    1970-04-04 05:30:00           66
    1970-04-04 01:30:00           64
    1970-04-04 01:15:00           63
    1970-04-01 23:30:00           60
    1970-04-04 05:00:00           60
    1970-04-04 04:45:00           59
    1970-04-04 05:15:00           59
    1970-04-04 01:45:00           56
    1970-03-13 22:15:00           55

Next I wanted to find the daily trends, so I pulled the time, month and day of month into their own columns, and plotted each day on top of eachother.  
![Daily Cycle of Logins](https://raw.githubusercontent.com/claireramming/Springboard-Mini-Projects/master/ultimate_challenge/imgs/dailycycle.png)

There is a clear login peak around noon and another leading up to midnight. There is also somewhat of an early morning peak, maybe this is where weekends get ahead in login counts. I decided to confirm this by aggregating the data into 3 hour chunks and plotting the 3am-6am chuck by day. I only chose to plot one month here, but you can clearly see 2-day peaks that correspond with weekends (recall Jan 1st was a thursday, so Jan 3/4 are weekends).  
![Early Morning Logins](https://raw.githubusercontent.com/claireramming/Springboard-Mini-Projects/master/ultimate_challenge/imgs/earlylogins.png)

### Part 2 - Experiment and Metrics Design

_The neighboring cities of Gotham and Metropolis have complementary circadian rhythms: on weekdays, Ultimate Gotham is most active at night, and Ultimate Metropolis is most active during the day. On weekends, there is reasonable activity in both cities._  

_However, a toll bridge, with a two-way toll, between the two cities causes driver partners to tend to be exclusive to each city. The Ultimate managers of city operations for the two cities have proposed an experiment to encourage driver partners to be available in both cities, by reimbursing all toll costs._ 

_1. What would you choose as the key measure of success of this experiment in encouraging driver partners to serve both cities, and why would you choose this metric?_

 The key meausure for this experiment would be toll usage. Due to the circadian rhythms of the city affecting   activity during the week, we'd want to track which toll is being used (into gotham or into metro), the day, and time  of day. 
  
_2. Describe a practical experiment you would design to compare the effectiveness of the proposed change in relation to the key measure of success. Please provide details on:_  
 _a. how you will implement the experiment_  
 
 Track toll-bridge usage, and we'd expect to see an increase in toll-bridge use.  
 
 _b. what statistical test(s) you will conduct to verify the significance of the observation_  
 
 If we have previous data, and we track this over several months, then we can do some bootstrap tests to see if there is a significant increase in toll usage in three areas:  
 1. During the Day into Metropolis  
 2. During the night into Gotham  
 3. weekends in general for both toll directions
   
 _c. how you would interpret the results and provide recommendations to the city operations team along with any caveats._
   
 If we are not seeing an increase in any area, then the system isn't working at all. If we are seing an increase during the week for either toll then we know at least drivers are no longer discouraged from being available when the cities are most busy. If there is no uptick on the weekends this may mean they are just busy enough in their own city that they don't need to cross, which may be fine. If we see an increase in all areas, then the plan seems to be working. Since we are only tracking toll usage, this could mean personal cars are using the tolls more frequently, we'd either need a way to only look at taxis, or cross-reference with the toll reimbursements.
 
### Part 3 - Predictive Modeling
 
_Use the given data set to help understand what factors are the best predictors for retention, and offer suggestions to operationalize those insights to help Ultimate._

I loaded the data with json:  

	import json 
	data = json.load(open('ultimate_data_challenge.json'))
	riders = pd.DataFrame(data)
	riders.info()
	
	<class 'pandas.core.frame.DataFrame'>
	RangeIndex: 50000 entries, 0 to 49999
	Data columns (total 12 columns):
	avg_dist                  50000 non-null float64
	avg_rating_by_driver      49799 non-null float64
	avg_rating_of_driver      41878 non-null float64
	avg_surge                 50000 non-null float64
	city                      50000 non-null object
	last_trip_date            50000 non-null object
	phone                     49604 non-null object
	signup_date               50000 non-null object
	surge_pct                 50000 non-null float64
	trips_in_first_30_days    50000 non-null int64
	ultimate_black_user       50000 non-null bool
	weekday_pct               50000 non-null float64
	dtypes: bool(1), float64(6), int64(1), object(4)
	memory usage: 4.2+ MB
	
From the infoo output, we can see there are null values in 3 of the columns. We will look at those a bit later. For now let's get a classification column. I updated the last_trip_date column to be a datetime object and found the most recent date in the column then subtracted thirty days from it and called it _lastday_. A rider is considered retained if they had a ride after lastday. 
	
	lastday = riders.last_trip_date.max() - pd.Timedelta('30 days')
	riders['retained'] = riders.last_trip_date > lastday
	
We can then get our retention rate with `len(riders[riders.retained == True])/len(riders.retained)*100` which gives us 36.62 %

I also changed the sign up date column to just contain the day of signup, since they all signed up during the month of January. 

Since this is a classification problem, I plotted the categorical features as bar plots, and numerical features as violin plots. I prefer violin plots over boxplots as they give a better sense of distribution.

I was most interested in the city and phone breakdowns. King's Landing was the only city that had a higher count of retained users than non-retained. On the phone breakdown, iphone and android users both had a higher non-retained count, but the ratio was much higher for android users. I did not include the nulls in my visual analysis since they could represent numerous things and I don't want to make incorrect assumptions. 

![city breakdown](https://raw.githubusercontent.com/claireramming/Springboard-Mini-Projects/master/ultimate_challenge/imgs/city.png)
![phone breakdown](https://raw.githubusercontent.com/claireramming/Springboard-Mini-Projects/master/ultimate_challenge/imgs/phone.png)

For the `avg_rating_(by/of)_driver` columns, I felt it appropriate to set the nulls to 0, since the lowest rating a driver or rider can give is 1, so setting them to 0 is a good way of keeping them in the data set but still seperating them from the actual ratings. Riders are more likely not to give a rating than drivers, especially if they are not retained. Retained riders seem to have more <4.5 ratings than non-retained, but this is most likely due to retained riders being more likely to take more than 2 rides in their first 30 days, leading to a greater chance of being rated multiple times and potentially getting a few 3's or 4's to lower their score from a 5. 

![rating of driver](https://raw.githubusercontent.com/claireramming/Springboard-Mini-Projects/master/ultimate_challenge/imgs/rating_of_driver.png)
![rating by driver](https://raw.githubusercontent.com/claireramming/Springboard-Mini-Projects/master/ultimate_challenge/imgs/rating_by_driver.png)
![rides in first 30 days](https://raw.githubusercontent.com/claireramming/Springboard-Mini-Projects/master/ultimate_challenge/imgs/ridesinfirst30days.png)

I decided to go with a Random Forest Classifier for my model. Random Forest gives a sense of the most important features in the model and should do better than logistic regression since our features aren't strongly correlated to our 'retained' data.

I got dummy variables for the categorical features city and phone. I dropped a city for the city metric, but did not drop any columns for phone since a 0 in both columns can represent our null values without introducing NaNs. 

With no other feature modification, I ran a test of random forest and got a ~.776 accuracy score. To get a sense of how good my model was and get a sense of the ideal number of estimators I used accuracy score over a range of n_estimators (10 to 300 in increments of 10).

![first run](https://raw.githubusercontent.com/claireramming/Springboard-Mini-Projects/master/ultimate_challenge/imgs/avg_rating_normal_avg_dist_normal.png)

I then looked at the most important features (using the `feature_importances_` attribute from sklearn) and found that the top three features were Avg Distance, Average rating by driver, and weekday percentage. 

I decided to focus on modifying these features to see if I could get an improvement in accuracy. Average rating by driver was particularly heavy in the 4.7-5 range, so I decided to cube that metric to give it some breathing room. That seemed to push the accuracy in the higher n_estimator range (>150) close to .780. I attempted to do a similar method with average distance, taking the log instead since the data leaned toward the lower end (less rides). Taking the log did not seem to help model performance, the accuracy curve for varying number of estimators responds slower with the change, so I ended up not keeping that in and resorting to the original data for avg dist. You can see this in the below graphs.

![first run](https://raw.githubusercontent.com/claireramming/Springboard-Mini-Projects/master/ultimate_challenge/imgs/rating_changed_dist_normal.png)
![first run](https://raw.githubusercontent.com/claireramming/Springboard-Mini-Projects/master/ultimate_challenge/imgs/rating_changed_dist_changed.png)

I used accuracy to get a good sense of how my model was doing, but the false positive rate is also an important metric for this case. We want to focus on the true negatives and false positives, a negative indicates a user will most likely not be retained, we want to limit our false positives, since we will be focusing our efforts on getting any predicted negatives to stay. False negatives are not as important since any effort we make will only increase their chances of sticking around, even if it is most likely that they were going to anyway. Here is the confusion matrix from the original model with no feature changes:  

	[8069, 1454]  
	[1906, 3571]    

Here is the confusion matrix from the model with the avg rating by driver cubed (no change to avg distance):

	[8087, 1436]  
	[1885, 3592]

A small improvement (took the false positive rate from .152 to .151), but still an improvement.

Now that we have a model, Ultimate can use this to increase their retainment rate. Users predicted negative could receive special offers that may convince them to stay. Even though the model did not measure it as most important, King's Landing is the only city with more users retained then lost, so focusing on that city for further analysis to see where other cities could improve and/or targeting discounts to predicted negatives in those cities could also be a next step for ultimate. Finally, there are a lot less android users than iphone users, and android users have a much lower retention rate. This may indicate the android application has bugs or UI issues. 
 

