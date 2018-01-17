###Relax Challenge Write-up

I created a Random Forest Classification model to predict user adoption, and found the most important factors in predicting user adoption is last session creation time and organization ID. Last session creation time makes sense because if a user has used the system recently there is a good chance they are using it frequently. 

![last session creation time](https://raw.githubusercontent.com/claireramming/Springboard-Mini-Projects/master/relax_challenge/imgs/sessioncreation.png)

Organization ID was somewhat surprising, I noticed after plotting a violin plot, the lower org IDs had a tendency to not have adopted users. I wouldn't expect real life data to pan out this way, but with proper manipulation (resorting organizations by percentage of adopted users weighted by number of users perhaps) it may be possible to mimic what was naturally occuring here.

![organization ID](https://raw.githubusercontent.com/claireramming/Springboard-Mini-Projects/master/relax_challenge/imgs/org_id.png)

I created some features that didn't end up having strong importance in the model. One is the email domain, I pulled that from the e-mail column. Some email domains were just a random string of letters so I grouped those into an 'other' category. I also created a feature for whether the user was invited by another user (this is seperate from an account created from a guest invite), and another for whether the user was invited by an adopted user. I ended up using the 'invited by an adopted user' feature in my model, thinking maybe users invited by adopted users would be more likely to become adopted users themselves. I think the lack of adopted users (adoption rate was only 16.7%) may have hindered this from being more useful of a metric.  

I began with logistic regression, but noticed by looking at the confusion matrix that my model was simply predicting 'False' for everything. Switching to Random Forest gave me a final accuracy of 90% once I plotted accuracy vs number of estimators to find a good estimator value (I chose 100). 

![Accuracy curve](https://raw.githubusercontent.com/claireramming/Springboard-Mini-Projects/master/relax_challenge/imgs/randomforestacc.png)

	confusion matrix:
	[2894,  114]
	[ 246,  346]

Since last session creation time ended up being so important, further exploration of the account creation time may be beneficial.  