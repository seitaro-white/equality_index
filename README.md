# Equality Data - Anaylsis and Index construction

This readme outlines following steps I have taken to construct an index from the company data provided:

1. Cleaning
2. EDA (Exploratory Data Analysis)
3. Missing Values
4. Weighting & Aggregation

The text is taken straight from markdown in the `equality_index.ipynb` - it probably makes more sense to read the notebook which has the code alongside.

The index itself can be found in the final cell of `equality_index.ipynb`


## Cleaning
***

### **Comparing March and June datasets**
- The June and March datasets have a slightly different list of companies, so this is actually an index of 103 companies when the two are combined
- The difference between March and June datapoints are generally small (or none), with some exceptions e.g. Barret Developments Gender Pay Gap jumping from 3.2% to 34.2%
- "Directors who are female (%)" and "New directors appointed who are female (%)" both have unexpectedly dramatic % changes. This is because of a discrepancy in the datasets - for March, the figures are percentages while for June they are decimals

### **Combine March and June datasets**
There are a number of different ways we could combine the June and March dataset into a single dataset 
- If this is a "live" index (e.g. gets updated once a month or even more frequently), it would make sense to primarily use the June dataset as representing the most up-to-date data, and only use the March data to forward-fill where data for June does not exist
- If this is a one-off index, or something that is only updated yearly, it might make more sense to average between months, particularly if some indicators vary significantly over time

For this project, I've kept things simple and used the first approach, using data-points from March to fill in missing values in June only. I haven’t done anything to change the March values, like trying to predict changes/forecast trends from March to June as the time difference is relatively small

## EDA
***
### **Visualise distribution of each indicator**

I visualised the distribution of each indicator to get a clearer view of what was going on (results in `/graphics`)

- The indicators essentially look mostly as expected - they are either continuous percentages or percentage changes
- "WISE Membership" is binary
- "Level of WISE membership" is ordinal
- "Gender bonus pay gap" has a long left-tail, which required some transformation for normalisation later (There were other indicators with tails but this one was the most extreme)

## Missing Values
***
### **Imputation Approach**
The basic story is that we generally have pretty good data coverage for indicators related to Gender, but much sparser data for the other indicators, particularly those related to LGBT+ and Disability. Two indicators (LGBT+ and Disability Pay Gap) hold no data at all.

There are a number of different approaches we could take to deal with these missing values, ranging from the very simple (replacing gaps with 0 or the lowest scoring value available), to trying to populate missing values with an average taken from other companies for that indicator, to more complex methods that try to model/predict a missing value using existing data elsewhere (e.g. Linear models or clustering techniques)

For this project, I’ve keep things simple, and proceed on the basis of the following assumption, which is that ***missing datapoints represent a failure by the responsible company to report, and it is therefore fair to penalise those companies by imputing 0 or lowest-scoring values***. Missing data is a non-ignorable non-response

The inverse of this is perhaps clearer - it would be *unfair* to penalise Company A (which has reported a below-average value for a given indicator) in relation to Company B (which may have similarly poor performance but has failed to report) by imputing Company B an averaged value and therefore giving them a "benefit of the doubt" advantage. This could potentially incentivise poorly performing companies to deliberately withhold reporting if they knew that they would be "pulled up" by the average in the index.

Based on this, I’ve taken the following approaches (applying them to each indicator on a case-by-case basis):

- Imputation by Zero
- Imputation by worst relative value

## **Normalisation**
*** 
I've simply normalising all indicators to a Min-Max scale of 0-1. The only other steps for some indicators taken were to:
- Invert indicators where a higher value is considered bad and vice versa
- Transform the "Gender Pay Bonus Gap (%)" indicator with the long tail

## **Weighting and Aggregation**
***

In working out a weighting scheme for the indicators, I've broadly considered two different methodologies/frameworks.

1) The impact of weighting from a purely statistical sense - thinking about the data isolated from any external factors, how can we weight our indicators to get a result that is statistically sound

2) The impact of weighting in a wider social context - thinking about the index as something that will be made publicly available, used by others and iterated over time, what value judgements are we making about Gender/Race/LGBT+/Disability by assigning certain weighting schemes? How might our index affect company behaviour as they seek to improve their rankings?

### **Weighting by Relevance**

Weighting by Relevance is about considering how directly relevant indicators are to determining a company's performance on equality. For this small project, I will keep my approach to weighting very simple and use just two relevancy tiers - indicators that I consider to be "fully relevant" and those that I consider to be only "partially relevant". The majority of indicators will be taken as fully relevant and given an equal weighting, but I consider these indicators to be only partially relevant for the following reasons:

- **WISE membership** and **Level of WISE membership** - It's possible for Companies to join organisations like WISE as a window-dressing exercise. WISE is also focused on women-in-stem, which may not necessarily apply to every company on our list

- **Gender pay gap improvement (% change)** - Pay gay improvement is relative. Company A with a pay gap of 30% but an improvement of 7% is still worse comparatively than Company B with a pay gap of 2% but an improvement of 0.5%

I did also consider **Employees who are female (%)**, because this is not necessarily informative about the state of gender equality on a company on it's own. A company could have majority female employees but also have a large pay gap, and all its senior positions occupied by men. However, I decided that **Employees who are female (%)** was useful in the negative sense - while a high percentage does not necessarily indicate good gender equality around pay/power within a company, an unusually low value is a definite indicator that something is wrong!

For indicators deemed "fully relevant", a weighting of 1 will be assigned, while for indicators deemed "partially relevant", a weighting of 0.5 will be assigned

### **Weighting by Evidence**

Weighting by Evidence is about weighting indicators according to how complete the data is - an indicator with less missing values should be considered more reliable and weighted accordingly. Weighting by Evidence causes us all sorts of problems here because of the nature of our dataset, which is heavily imbalanced towards Gender - I'll provide two alternative approaches and discuss the benefits and draw backs of each to explain this

1) One approach would be to weight each indicator in direct proportion to the completeness of the reported data - for example, "Gender pay gap (%)" might be weighted at 0.87, and "BAME pay gap (%)" at just 0.03. This is attractive from a purely statistical point of view for obvious reasons. However in doing so, we are making an implicit value judgement about which factors matter for equality - since the majority of reported data we have is for Gender indicators, we are saying that Gender is therefore much more important than Race, LGBT+ or Disability in measuring equality within a company which is not necessarily a desirable outcome (to be precise for this dataset, Gender would make up 93.5% of the index) In addition, assigning low weight indicators with low reporting could lead to a vicious circle - if disability indicators are assigned very low weight because of the lack of data, companies seeking to improve their rank in future will have very little incentive to actually improve reporting around disability, meaning that in the next iteration reporting will be similarly poor and so on.

2) An alternative approach would be to ignore evidence entirely and simply consider each indicator equally regardless of reporting completeness. An even further step might be to take an "intersectional" approach and combine all indicators into 4 pillars for Gender/Race/LGBT+/Disability and weight each pillar equally as 25% of the index. The obvious problem here for our dataset is that we would end up with an index where 93.5% of the actual reported data ends up informing just 63% of the final rank, or only 25% with the intersectional approach, which seems to defeat the point of researching an index all together. 
    (An additional problem is that data collection/reporting on the different equality dimensions can't be treated as exactly identical -  LBGT+ and Disability particularly are sensitive areas where employees may feel unwilling or unable to disclose information about themselves for reasons both internal and external to a company)

For this project, I've decided to take a hybrid approach that represents a compromise between the two extremes - I'll treat relevancy and evidence as two entirely separate components of weighting for each indicator that are calculated separately and averaged at the end. Since as mentioned earlier relevancy is almost always equal, this will act as a baseline weighting for each indicator. For example, for the earlier example of Gender vs BAME pay gap:
- They are both assigned relevancy weight of 1
- They are assigned an evidence weight of 0.87 and 0.03 respectively
- Relevancy and Evidence weights are averaged (by mean)
- The total sum is 0.935 for Gender, and 0.515 BAME - the BAME pay gap is weighted lower due to lack of evidence, but not *so* much lower that it becomes insignificant

It should be acknowledged this compromise approach is less that ideal, but hopefully it does avoid the worse extremes of the 2 initial approaches explained above
