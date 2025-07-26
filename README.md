# Evaluating Offensive Linemen Using OLIZ: Offensive Linemen Immediate Zone

Note: If running yourself, you will need data from the NFL's 2023 Big Data Bowl. The correct order to run the .ipynb files on your machine is as follows: Data_Preparation, Feature_Creation, Model_Creation. These will take you through all of the steps to this project. Exploratory_Analysis was used to brainstorm calculations and build features.

Authors: Peter Majors, Chris Orlando, Etienne Busnel

**<font size = "5" color = 'maroon'>Introduction**

<font size = "4"> When a quarterback drops back in the pocket, what is the role of the offensive lineman? Generally, it's to keep the pass rusher as far from the quarterback as possible. Doing so, he gives the quarterback enough time, physical space, and peace of mind to complete a pass to his receiver.

<font size = "4"> Below we see a series of combinations between distances of various numbers of pass rushers from a quarterback on the final frame he has possession of the football, along with the associated Expected Points Added (EPA) of the play. We see that as the distance decreases and and the number of rushers within that distance increases, offensive performance declines.

![image](https://user-images.githubusercontent.com/73561125/212229770-f89c2b8c-321f-4796-af13-6c6aed9d9aa6.png)

<font size = "4"> Even the most casual of football fans are aware of the relationship between pressure on the quarterback and offensive performance. We aren’t breaking any new ground here. However, what keeps these pass rushers from reaching the quarterback? Well, that would be a series of well fed offensive linemen, patrolling the center, tight end, and guard positions. 
    
<font size = "4"> Classic approaches to offensive linemen performance concern themselves with binary outcomes such as pressures and sacks, as they are easy to understand, but these outcomes can be difficult to predict and subject to human biases. Among pass blockers with 100 snaps in weeks 1-4 and 5-8, hurries allowed and sacks allowed had a correlation of <b> .32 </b> and <b> .25 </b>, respectively, for the first and second four weeks of the 2021 season. While sacks are hurries are high impact plays in terms of EPA, their predictive value is suspect. 
    
<font size = '4'> With this in mind, we decided to investigate, not sacks nor hurries, but a something present in nearly every play: **rusher distance from the quarterback at the end of posession**, and its impact on EPA as well as predicitve value for sacks and hurries.

**<font size = "5" color = 'maroon'> Our Proposal**

<font size = "4"> We set out to determine the performance of offensive linemen as it pertains to their ability to keep the pass rusher as far as possible from the quarterback at the final moment of the quarterback’s possession of the football. Before proceeding, let’s lay the groundwork for our analysis. Below are the considerations used to determine which plays were selected for our analysis.
    
- <font size = "4"> All Passing Plays, First 8 Games Of The 2021 NFL Season
- <font size = '4'> First Block By Each Pass Blocker Against A Known Pass Rusher
- <font size = '4'> Excluding Block Types “Backfield Help”, “Chip Block”, “No Block”, and “Set & Release”
- <font size = '4'> Excluding Non-Traditional Quarterback Drop Backs (Designed Rollouts, Scramble Rollouts, etc.)
- <font size = '4'> Blocks Performed By Left Tackles, Left Guards, Centers, Right Guards, and Right Tackles 
- <font size = '4'> Time Between Ball Snap And End Of QB Posession (Ball Release, Run, Strip Sack, Sack)
    
<font size = '4'> To determine when an offensive lineman was engaged with their pass rusher, we created a orientation responsive “immediate zone” in front of each pass blocker, designated as their <b> OLIZ </b>, or <b> “Offensive Lineman Immediate Zone” </b>. The OLIZ shifts with the pass blocker’s shoulder orientation at each frame, reaches 1.5 yards in front of them, and spans 1 yard to their left and right. We found that 1.5 yards is the distance in front of a pass blocker where the momentum (speed and acceleration) of an oncoming pass rusher is lowest.

![OLIZ_demonstration_hd](https://user-images.githubusercontent.com/73561125/212230097-a70bd37b-0645-4469-88ca-b5bcdb193069.png)

<font size = "4"> OLIZ focuses on the first rusher assigned to each pass blocker in a play, based on data that only tracks the initial block performed by each player. It does not capture double teams or their effects. Using OLIZ, we developed spatio-temporal metrics to assess how well an offensive lineman contains a single pass rusher.

<font size = "4"> In creating the following questions and features, we aimed to isolate one-on-one matchups between blockers and rushers. Rather than predicting the rusher’s final distance from the quarterback, we concentrated on quantifying the offensive lineman’s individual influence in typical pass protection situations.

<font size = "5" color = 'maroon'> Questions Considered and Associated Features

- <font size = "4">**How well can the blocker hold their ground?**  
  - Rusher distance from QB at beginning and end of being in OLIZ

- <font size = "4">**How long can the blocker stay engaged with the pass rusher?**  
  - Time spent by the rusher in the OLIZ

- <font size = "4">**How long can the blocker stay between the QB and the pass rusher?**  
  - Blocker time spent closer to QB than the rusher

- <font size = "4">**What is the direction of the rusher relative to the orientation of the blocker coming into the block?**  
  - Diff between rusher direction and blocker orientation when rusher enters the OLIZ

- <font size = "4">**How well can the blocker maintain similar shoulder orientation to the rusher?**  
  - Diff in rusher and blocker shoulder orientation at beginning and end of OLIZ

- <font size = "4">****How well can the blocker slow the rusher as they're leaving the immediate zone?**  
  - Speed of the rusher leaving the OLIZ

- <font size = "4">****If the blocker loses control of the pass rusher, how well can they recover?**  
  - Rusher time spent in OLIZ after leaving for the first time
  
<font size="4">If the Blocker Loses Control of the Pass Rusher, How Well Can They Recover?
  - Total rusher time spent in the OLIZ after leaving it for the first time

<font size="5" color="maroon">Modeling

We trained two separate models: one for interior linemen (guards and centers), and another for tackles. We used Extreme Gradient Boosted (XGBoost) Regression, a robust ensemble method well-suited for capturing non-linear interactions and feature importance, to predict the distance between the rusher and the quarterback at the end of the play. We used StandardScalar() to account for differences in magnitude and K-Fold Cross Validation to tune hyperparameters - paying special attention to overfitting.

![feature_importance](https://user-images.githubusercontent.com/73561125/212230425-882a0527-559a-4d28-ab3c-7be56acc735c.png)

<font size = "4"> As you can see, interior and exterior offensive linemen respond quite differently to the selected features. 
    
<font size="4"> For guards and centers, the most influential feature was the change in rusher distance from the quarterback between the start and end of their time in the OLIZ. This aligns with expectations—interior linemen often face immediate pressure, so their ability to hold ground is critical in pass protection. We also found that the duration a rusher remained in the OLIZ played a significantly larger role for guards and centers than it did for tackles, further emphasizing the importance of sustained engagement in the interior. On the test set for guards and centers, the model achieved a <b> RMSE of 1.27 yards </b> and a <b> Pearson correlation of 0.72 </b>. This indicates that approximately 52% of the variation in a rusher’s final distance from the quarterback can be explained by their initial interaction with the interior offensive lineman.

<font size="4"> For tackles, the most important feature remains the change in rusher distance from the start to the end of the immediate zone. However, two additional factors stand out: the amount of time the blocker stays positioned between the quarterback and the rusher, and the time the rusher spends back in the immediate zone after initially leaving it. These metrics highlight the critical importance of maintaining positioning and regaining control during pass blocking—a finding now supported by our model. On the test set for tackles, the model performed strongly, achieving an <b> RMSE of 1.13 yards </b> and a <b> Pearson correlation of 0.81 </b>, indicating a high level of accuracy in predicting rusher distance outcomes.

**<font size = "5" color = 'maroon'> Evaluation**

<font size = "4"> On a team-by-by team basis, we see very promising results when comparing actual and predicted rusher yards from quarterback at the final moment the play. This may Below, the guards/center and tackle scatterplots show a <b> .62 </b> and <b> .71 Pearson R </b> correlation, respectively. Being towards the top-right is better, since that means their offensive linemen prevented rushers from being closer to the QB.
    
<font size = "4"> Note: The below graphs only include predictions on Weeks 5-8 of the data from the 2021 NFL season.

![dist_qb_pred_teams](https://user-images.githubusercontent.com/73561125/212230519-229ccdda-4f60-40d8-a97d-016eeb5542d6.png)

<font size = "4"> Below we see the top 10 guards, rushers, and tackles at keeping rushers away from their quarterabcks in Weeks 5-8 of the 2021 NFL season, as predicted by their performance in Weeks 1-4. As you can see, these aren't the consensus "best" offensive linemen in their position groups - they are simply the ones we projected to provide the QB space at the time of release.

![dist_qb_allowed_players](https://user-images.githubusercontent.com/73561125/212230600-e7c6add5-31dd-4bd6-85e6-cfdb112743b8.png)

<font size = '4'> Despite the predictive power of model - in a four week sample, statistics and matchups can be volatile. As for considerations about varying rusher talent levels, we attempted to capture the 'talent' of these rushers within each play by using our spatio-temporal features. There are also considerations about QB elusiveness. As mentioned above, we intentionally limited our sample to plays where QBs stayed in the pocket - but controlling for this variable, especially as team change QBs would have improved model reliability.

<font size = '4'> Below, we observe that our model performs noticeably better than predicting distance from the QB with sacks, hurries, or the target variable itself from Weeks 1-4 of the season - showing promise for future applications.

![dist_qb_other_metrics](https://user-images.githubusercontent.com/73561125/212230624-fcc1ca5a-45a9-4755-8277-e431bc4b5ec0.png)

**<font size = "5" color = 'maroon'> Conclusion**

<font size = '4'> In order to make this analysis feasible, we had to eliminate many of the factors that make play on the offensive line so difficult to quantify, including scrambles and comlex schemes. This is one of the unfortunate realities of working with o-line data - and football data as a whole. Some areas of improvement for our model include optimizing the shape of the OLIZ, dynamically sizing the OLIZ based other teammates' zones, detecting/accounting for double teams, controlling for QB elusiveness in the pocket, and expanding our feature set to measure more about the oncoming pass rusher.

<font size = '4'> By relying solely on player tracking data, we developed a model that more accurately predicts the distance a rusher is kept from the quarterback—an objective, data-driven measure of pass protection effectiveness. For interior offensive linemen, our model explains approximately half of the variation in this outcome. For tackles, that figure rises to two-thirds, underscoring the model’s strong performance in capturing the nuances of edge protection. We hope this project contributes, even in a small way, to the broader effort of advancing offensive line evaluation—shedding light on the specific, measurable actions that distinguish high-performing linemen and pushing the field toward smarter, more transparent player assessment.
