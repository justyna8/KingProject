

Assumptions: The test is clean. Since the version A is stend standard, somebody might had been exposed to it before being assigned to test group B which would lead to cross contamination of the test groups. I do have an installation date however I do not have "first usage" date which means I cannot take this into account. I will therefore assume that (1) only using data after the test (2) the new experiance is unparalled and unique enough that being exposed to the original BAU contaminates the experience in group B. 


The question: how would we determine which of the experiences make Super Math Saga a better game?

The first question to be answer is what is a qualification of a better game? Who is the game better for? If the game is better for the player then it should have that  very right level of difficulty that provides an adequate challange but is also not absolutely impossible to solve without having to result to a purchase. In order to find the most optimal experiance for the user I would use `gameends` as a KPI. High amount of game ends indicates that the user enjoyed themeslevs and meant to keep on playing. A better experiance for the company is the one where the user keeps on playing (and the feature does not distract from the the game) but also chooses to make a purchase. Therefore the most optimal version of the game would be a mix of higher `purchases` and higher `gameends`.

>As there might not necessarily be a correlation between game plays and purchases >(more purchases result in more game ends but the player might sooner hit the amoutn >they are willing to pay)


>• Better game for the company: which one is bring the most revenue?
>(istalling vs first purchase) Did the person make a purchase?
>(intalling vs purchase date) What was the amount of time between installing and >making the first purchase? 
>Did the make the first purchase after being assigned?


```sh
project <- "king-ds-recruit-candidate-43"
library(bigrquery)
library(readr)
GamePlays <- "SELECT assign.abtest_group, sum(ac.gameends) as SUMGameEnd, count(distinct(ac.playerid)) as Players, sum(ac.gameends)/count(distinct(ac.playerid)) as ratio FROM [king-ds-recruit-candidate-43:abtest.activity]  ac
+ INNER JOIN  [king-ds-recruit-candidate-43:abtest.assignment] assign
+ ON assign.playerid = ac.playerid  
+ WHERE ac.activity_date>=assign.assignment_date
+ AND ac.activity_date BETWEEN '2017-05-04' AND '2017-05-22'
+ group by 1 order by 1 LIMIT 100"
> GamePlayResults <- query_exec(GamePlays, project=project)
5.8 gigabytes processed                                                           
> GamePlayResults
  assign_abtest_group SUMGameEnd Players      ratio
1                   A 1309402096 8161682 160.4329
2                   B  321165708 2130557 150.7426
```
This calculation shows us that users average more `gameends` when they are following version A. There is a ~6% lift in game rounds played per person for the group A.


Looking at Purchases, another key metric. The version that bring higher revenue would 

```sh
Purchases <- "SELECT assign.abtest_group, sum(ac.purchases) as Purchases, count(distinct(ac.playerid)) as Players
+ ,sum(ac.purchases)/count(distinct(ac.playerid)) as ratio
+ FROM [king-ds-recruit-candidate-43:abtest.activity]  ac
+ INNER JOIN  [king-ds-recruit-candidate-43:abtest.assignment] assign
+ ON assign.playerid = ac.playerid  
+ WHERE ac.activity_date>=assign.assignment_date
+ AND ac.activity_date BETWEEN '2017-05-04' AND '2017-05-22'
+ group by  assign.abtest_group, order by 2 DESC"
> PurchasesResults <- query_exec(Purchases, project=project)
> PurchasesResults
  assign_abtest_group Purchases Players     ratio
1                   A   3026154 8161682 0.3707758
2                   B    776375 2130557 0.3644000
> PurchaseTest <- prop.test(x = c(3026154, 776375), n = c(8161682, 2130557))
> PurchaseTest

	2-sample test for equality of proportions with continuity correction

data:  c(3026154, 776375) out of c(8161682, 2130557)
X-squared = 294.79, df = 1, p-value < 2.2e-16
alternative hypothesis: two.sided
95 percent confidence interval:
 0.005649237 0.007102293
sample estimates:
   prop 1    prop 2 
0.3707758 0.3644000 
```

Group A perfomrd better when it comes to purchases as well. Experiance B had players not only play less rounds but also be less likely to make a purchase, thus likely bringing less revenue to the company. This is likely because experiance B (1) is too easy The player feels neither compelled to keep on playing, since they are not challanged enough; they also never find a need to make a purchase or (2) and/or boring. This makes the players drop off ona  higher rate, and lead to higher drop off in purchases.

Look at daily installs and assignments over the duration of the A/B test. Do you expect to see any difference between groups?
I am expecting to see difference between the two groups as assignments happened in bulk on one day while installs have likely been happening for some period of time, and unless an outside factor impacted them (for example, a promotion) they are liky to be constant. We can clearly tell that if we graph both groups as well as p-value over .05:
```sh
AssignDates <- "SELECT count(assignment_date)
+ FROM [king-ds-recruit-candidate-43:abtest.assignment] 
+ WHERE assignment_date BETWEEN '2017-05-04' AND '2017-05-22'
+ group by assignment_date order by assignment_date"
> AssignDatesREsults <- query_exec(AssignDates, project=project)
> boxplot(AssignDatesREsults)
```
![N|Solid](https://image.ibb.co/iz10Uy/Assign_Dates_REsults.jpg)
```sh
InstallDates <- "SELECT count(install_date)
+ FROM [king-ds-recruit-candidate-43:abtest.assignment] 
+ WHERE install_date BETWEEN '2017-05-04' AND '2017-05-22'
+ group by install_date order by install_date"

> InstallDatesResults <- query_exec(InstallDates, project=project)
> boxplot(AssignDatesREsults)
```
![N|Solid](https://image.ibb.co/d7mH9y/Install_Dates_Results.jpg)
```sh
t.test(InstallDatesResults,AssignDatesREsults)

	Welch Two Sample t-test

data:  InstallDatesResults and AssignDatesREsults
t = -1.0002, df = 18.001, p-value = 0.3305
alternative hypothesis: true difference in means is not equal to 0
95 percent confidence interval:
 -1381724.8   490415.3
sample estimates:
mean of x mean of y 
 98049.16 543703.95 
```

• What other aspects do you think would be valuable to consider for analysing an A/B test?



My recommendation would be to conclude the test, and implement experiance A for both groups. It has proven to not only lead to more game plays, thus bring higher revenue, but also bring more revenue for the company.  I would also recommend looking at the behaviour of the group B post test and see if there are any trends which would prove this group to be affected by the experince B. For example making 

