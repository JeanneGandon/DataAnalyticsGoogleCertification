**Introduction to the Case Study** <p>

You are a junior data analyst working on the marketing analyst team at Bellabeat, a high-tech manufacturer of health-focused
products for women. Bellabeat is a successful small company, but they have the potential to become a larger player in the
global smart device market. Urška Sršen, cofounder and Chief Creative Officer of Bellabeat, believes that analyzing smart
device fitness data could help unlock new growth opportunities for the company. You have been asked to focus on one of
Bellabeat’s products and analyze smart device data to gain insight into how consumers are using their smart devices. The
insights you discover will then help guide marketing strategy for the company. You will present your analysis to the Bellabeat
executive team along with your high-level recommendations for Bellabeat’s marketing strategy<p>

**Data Analysis Process:**<p>

### [Ask](#1-ask)
### [Prepare](#2-prepare)
### [Process](#3-process)
### [Analyze](#4-analyze)
### [Share](#5-share)
### [Act](#6-act)

## 1. ASK
**Business Task**: Guide marketing strategy for Bellabeat by analyzing data on consumer usage and habits of other popular smart device. <p>
**Primary Stakeholders**: <br>
- Urška Sršen : co-founder and CCO <br>
- Sando Mur : co-founder and Mathematician <br>

**Secondary Stakeholders**: <br>
- Marketing Analytics team: responsible for collecting, analyzing, and reporting data to guide marketing stragetegy<p>
## 2. PREPARE

**Data Source**: <br>
- 30 participants FitBit Fitness Tracker data over 2 months
- Survey conducted by Amazon Mechnical Turk
- 18 CSV date file
- Data available on the second, minute, hour, and day levels
- Data available for heart rate, burnt calories, activity intensity, MET, steps, and sleep acitivy metrics <br>

**Data Assessment**: <p>
Reliability: **LOW** : very small sample size of 30 participants <br>
Originality: **LOW** : third party data <br>
Comprehensive: **MEDIUM** : comprehensive variety of metrics but most data is recorded from Tuesday to Thursday <br>
Curent: **LOW** : As of today, June 2023, data is 7 years old <br>
Cited: **HIGH** : Public domain <br>

Other limitations: Unknown demographics of participants while Bellabeat manufactures products exclusively for women.<p>

## 3. PROCESS
**Data used** : Daily Activity, Daily Intensities, Heart Rate, and Sleep. <p>
**Why?** : Bellabeat provides users with health data related to their activity, sleep, stress, menstrual cycle, and mindfulness habits. Daily Activity and Sleep are metrics that Bellabeat focuses on, and Stress can be assumed when the heart rate is high. <p>

**Data Import & Cleaning**: <p>
- Imported wanted data
        
```
activity <- read.csv("/kaggle/input/fitbit/Fitabase Data 4.12.16-5.12.16/dailyActivity_merged.csv")
heartrate <- read.csv("/kaggle/input/fitbit/Fitabase Data 4.12.16-5.12.16/heartrate_seconds_merged.csv")
sleep <- read.csv("/kaggle/input/fitbit/Fitabase Data 4.12.16-5.12.16/sleepDay_merged.csv")
```
- Examine data by checking for NA and duplicates and removing the ones found in the sleep data
```
sum(duplicated(activity))
sum(duplicated(heartrate))
sum(duplicated(sleep))
sleep <- sleep %>%
  distinct() %>%
  drop_na()
```
- Clean up data by converting the dates and times to a universal format
```
activity <- activity %>%
    rename(Date = ActivityDate) %>% 
    mutate(Date = as.Date(Date, format = "%m/%d/%y"))
heartrate <- heartrate %>%
    mutate(time = mdy_hms(Time)) %>% 
    separate(col = time, into = c("Date", "Datetime"), sep = " ")
sleep <- sleep %>%
    rename(Date = SleepDay) %>% 
    mutate(Date = as.Date(Date, format= "%m/%d/%Y  %I:%M:%S %p", tz= Sys.timezone()))
```

## 4. ANALYZE

### User Utilization 
```
activity1 <- unique(activity$Id, incomparables = FALSE)
sleep1 <- unique(sleep$Id, incomparables = FALSE)
heartrate1 <- unique(heartrate$Id, incomparables = FALSE)
# Plot
venn <- venn.diagram(x = list(activity1, sleep1, heartrate1),
  category.names = c("Daily Activity", "Sleep monitor", "Heart monitor"),
  filename = "features_venn.png",
  output=TRUE, imagetype="png",
  lwd = 2, fill = c("skyblue", "pink1", "orange"), 
  cex = 1, fontface = "bold", fontfamily = "sans",
  cat.cex = .7, cat.fontface = "bold", cat.default.pos = "outer", cat.fontfamily = "sans")
```

## 5. SHARE



## 6. ACT
