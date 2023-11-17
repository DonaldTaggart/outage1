# Exploratory Data Analysis on Outage Data
DSC 80 Project 3 | Donald Taggart

---
### Introduction

   The dataset this site analyzes covers major power outages across the US from January 2000 to July 2016. This analysis primarily focused on how quickly the WECC, California's main region, gets power back on relative to the entire country. The "raw" dataset has 1540 columns, but only 1534 each represent an outage. This raw dataset also consists of 57 columns. Important columns include:
- "U.S.\_STATE": Tells the state (including Washington D.C.)
- "NERC.REGION": Tells the North American Electric Reliability Corporation (such as the WECC)
- "ANOMALY.LEVEL": Tells the oceanic El Niño/La Niña index which refers to the cold and warm episodes (float values representing an estimate of a three-month running mean)
- "CAUSE.CATEGORY": Tells the general cause
- "CAUSE.CATEGORY.DETAIL": Tells a slightly more detailed description than "CAUSE.CATEGORY"
- "OUTAGE.DURATION": Tells how long the outage lasted (int values representing minutes)

---
### Cleaning and EDA

   The first step of preparing the data was making it neat from a pandas DataFrame persepective instead of that of Excel. This involved dropping rows that didn't represent an outage and turning on of the rows into the column labels.
   
   To focus only with data I would be working on for the plots and tests, I dropped a strong majority of the columns from the raw dataset. I also created a new boolean column of my own, "WECC", that would be a little simpler to work with down-the-line for some of the WECC-specific analysis than utilizing the initial "NERC.REGION" column.
   
   I also used a helper function that utilized pd.Timestamp() to convert four time-related columns into only two, but ended up dropping the resulting two columns because they were not needed for any of the analysis that was to come.
   
   I also removed any row where the "OUTAGE.DURATION" value was missing because it was unclear based off of the provided data and information available what type of missingness mechanism it resulted from (for example it wasn't evident that the missingess of "OUTAGE.DURATION" was related to values from columns such as "NERC.REGION"). These rows made up less than 4% of the initial data so it was a relatively trivial amount that wouldn't have had a sizable impact on future results even if it were NMAR or MAR
   
```py
outage_df.head()
```
   
| U.S._STATE   | NERC.REGION   |   ANOMALY.LEVEL | CAUSE.CATEGORY     | CAUSE.CATEGORY.DETAIL   |   OUTAGE.DURATION | WECC   |
|:-------------|:--------------|----------------:|:-------------------|:------------------------|------------------:|:-------|
| Minnesota    | MRO           |            -0.3 | severe weather     | nan                     |              3060 | False  |
| Minnesota    | MRO           |            -0.1 | intentional attack | vandalism               |                 1 | False  |
| Minnesota    | MRO           |            -1.5 | severe weather     | heavy wind              |              3000 | False  |
| Minnesota    | MRO           |            -0.1 | severe weather     | thunderstorm            |              2550 | False  |
| Minnesota    | MRO           |             1.2 | severe weather     | nan                     |              1740 | False  |
   
   Something I wanted to ensure by visualizing the data I had on hand was that there was a sufficient number of data points for both outages in the the WECC region, but also outside of it as well. The bar chart below displays how even though the WECC region experienced the most outages, it still only accounted for a strong minority of all outages, indicating that there was more than enough data to work with.
   
<iframe src="assets/outages_per_region_bar_chart.html" width=800 height=600 frameBorder=0></iframe>
   
   Before doing any official testing, I wanted to get a general idea of how outage durations compared across the various NERC regions. I formed the box plot below to get a general idea of whether or not the WECC region stood out from the others to any degree. It can be seen that even the WECC region has many outliars, it's mostly due to the fact that the WECC's max "whisker" doesn't reach quite as high as many of the other majors regions. In fact, it can easily be seen that its interquartile range is much lower than other large regions to the point that its visibly hard to distinguish the lines that indicate Q1, the median, and Q3. Its longest duration is also not quite as extreme as the those of a couple other major regions.
   
<iframe src="assets/outage_durations_per_region_box_plot.html" width=800 height=600 frameBorder=0></iframe>
   
   I wanted to inspect the WECC in particular because I wanted to analyze California's primary region. The pivot table below presents the percentage of each state's outages that were handled by a certain region. It can be seen that over 99% of California's outages occured in the WECC region. Eleven other states are also covered to varying degrees by the WECC.

```py
outage_pivoted_percents.head()
```

| U.S._STATE   |   ECAR |   FRCC |   FRCC, SERC |   HECO |   HI |   MRO |   NPCC |   PR |   RFC |   SERC |   SPP |   TRE |   WECC |
|:-------------|-------:|-------:|-------------:|-------:|-----:|------:|-------:|-----:|------:|-------:|------:|------:|-------:|
| Alabama      |      0 |      0 |            0 |      0 |    0 |  0    |      0 |    0 |     0 | 100    |     0 |     0 |   0    |
| Arizona      |      0 |      0 |            0 |      0 |    0 |  0    |      0 |    0 |     0 |   0    |     0 |     0 | 100    |
| Arkansas     |      4 |      0 |            0 |      0 |    0 |  0    |      0 |    0 |     0 |  56    |    40 |     0 |   0    |
| California   |      0 |      0 |            0 |      0 |    0 |  0    |      0 |    0 |     0 |   0.51 |     0 |     0 |  99.49 |
| Colorado     |      0 |      0 |            0 |      0 |    0 |  7.14 |      0 |    0 |     0 |   0    |     0 |     0 |  92.86 |

---
### Assessment of Missingness

   Without having more detailed information on how the data in each column was collected, it's hard to determine definitively if any of them are NMAR because it's unclear as to why any given value is unavailable and therefore missing. There is an argument to be made that "CAUSE.CATEGORY.DETAIL" is NMAR because its missingness depends on how much is known about the cause. For example, if no details were firmly known about the outage's cause, then whoever was entering the data may have decided to leave it empty instead of making an educated guess.
   
   The "NERC.REGION" column had no missing values and the "OUTAGE.DURATION" column's missingness was trivial so I decided to look at different columns to conduct permutation tests on.
   
   To figure out the possiblity that the missingness in the "CAUSE.CATEGORY.DETAIL" was related on the values in the "CAUSE.CATEGORY" column, I conducted a permuation test that involved the distribution of "CAUSE.CATEGORY" data when "CAUSE.CATEGORY.DETAIL" was missing and the distribution of "CAUSE.CATEGORY" data when "CAUSE.CATEGORY.DETAIL" wasn't missing. 1000 permutations of the "CAUSE.CATEGORY.DETAIL" led to a p-value of 0.0 because none of the permutations led to a sample that had a greater total variation distance than the observed tvd of about 0.42. This p-value heavily indicates that the "CAUSE.CATEGORY.DETAIL" column is MAR due its relation to "CAUSE.CATEGORY".
   
<iframe src="assets/tvd_empirical_distribution.html" width=800 height=600 frameBorder=0></iframe>
   
   Another permutation test showed that "CAUSE.CATEGORY.DETAIL" was independent of "ANOMALY.LEVEL" because the resulting p-value was 0.763.
   
---
### Hypothesis Testing

   My null hypothesis was that the distribution of "OUTAGE.DURATION" for WECC comes from the overall distribution whereas my alternative hypothesis was that the distribution of "OUTAGE.DURATION" for WECC is separate from the overall distribution. I chose the median of a region's outage duration as the test statistic because the data is numerical and it wouldn't be impacted too heavily by extreme duration outliars (which can be seen in the box plot in the Cleaning and EDA section). I went with a significance level of 0.05 as that is common in the industry and the resulting p-value of 0.0 is so far below said significance level that the null hypothesis can be rejected. Such a low p-value resulted from the oberserved test statistic of 219.5 being below all 10,000 sample test statistics.
   
<iframe src="assets/dims_empirical_distribution.html" width=800 height=600 frameBorder=0></iframe>