**Introduction to the Case Study** <p>

You are a junior data analyst working on the marketing analyst team at Bellabeat, a high-tech manufacturer of health-focused
products for women. Bellabeat is a successful small company, but they have the potential to become a larger player in the
global smart device market. Urška Sršen, cofounder and Chief Creative Officer of Bellabeat, believes that analyzing smart
device fitness data could help unlock new growth opportunities for the company. You have been asked to focus on one of
Bellabeat’s products and analyze smart device data to gain insight into how consumers are using their smart devices. The
insights you discover will then help guide marketing strategy for the company. You will present your analysis to the Bellabeat
executive team along with your high-level recommendations for Bellabeat’s marketing strategy. <p>

**Data Analysis Process:**<p>

### [Ask](#1-ask)
### [Prepare](#2-prepare)
### [Process](#3-process)
### [Analyze](#4-analyze)
### [Share](#5-share)
### [Act](#6-act)

## 1. ASK
**Business Task**: Guide marketing strategy for Bellabeat by analyzing data on consumer usage and habits of another popular smart device. <p>
**Primary Stakeholders**: <br>
- Urška Sršen : co-founder and CCO <br>
- Sando Mur : co-founder and Mathematician <br>

**Secondary Stakeholders**: <br>
- Marketing Analytics team: responsible for collecting, analyzing, and reporting data to guide marketing strategy <p>
## 2. PREPARE

**Data Source**: <br>
- 30 participants FitBit Fitness Tracker data over 2 months
- Survey conducted by Amazon Mechanical Turk
- 18 CSV data file
- Data available on the second, minute, hour, and day levels
- Data available for heart rate, burnt calories, activity intensity, MET, steps, and sleep activity metrics <br>

**Data Assessment**: <p>
Reliability: **LOW** : very small sample size of 30 participants <br>
Originality: **LOW** : third party data <br>
Comprehensive: **MEDIUM** : a comprehensive variety of metrics but most data is recorded from Tuesday to Thursday <br>
Current: **LOW** : As of today, June 2023, data is 7 years old <br>
Cited: **HIGH** : Public domain <br>

Other limitations: Unknown demographics of participants, while Bellabeat manufactures products exclusively for women.<p>

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
Produced a Venn diagram showcasing which metrics users are most likely to utilize:
```
activity1 <- unique(activity$Id, incomparables = FALSE)
sleep1 <- unique(sleep$Id, incomparables = FALSE)
heartrate1 <- unique(heartrate$Id, incomparables = FALSE)
# Plot
venn <- venn.diagram(x = list(activity1, sleep1, heartrate1),
  category.names = c("Daily Activity", "Sleep monitor", "Heart monitor"),
  filename = "features_venn.png",
  output=TRUE, imagetype="png",
  lwd = 2, fill = c("skyblue", "pink", "orange"), 
  cex = 1, fontface = "bold", fontfamily = "sans",
  cat.cex = .7, cat.fontface = "bold", cat.default.pos = "outer", cat.fontfamily = "sans")
```
<img src="https://github.com/JeanneGandon/DataAnalyticsGoogleCertification/assets/138037134/35869ab1-d1dc-4b27-a46d-ba0656ec2ab0" width=430> <br>
All users will use daily activity tracking, most will monitor their sleep, and fewer will monitor their heartbeat.

### Sleep Efficiency
With access to data on the total time asleep and the total time in bed, I calculated the sleep efficiency factor for each user per day:
```
sleepefficiency <- sleep$TotalMinutesAsleep/sleep$TotalTimeInBed
```
Was then able to graph the Sleep Efficiency Over Time for each user:
```
sleep$Id <- as.factor(sleep$Id)
ggplot(sleep, aes(x = Date, y = sleepefficiency, color = Id)) +
  geom_line() +
  labs(x = "Date", y = "Sleep Efficiency") +
  ggtitle("Sleep Efficiency Over Time") +
  scale_color_discrete(name = "Person")
```
![Screen Shot 2023-06-29 at 2 43 23 PM](https://github.com/JeanneGandon/DataAnalyticsGoogleCertification/assets/138037134/def303e6-80a7-481b-a6da-8dbfb4620b8a) <br>
Most users lie around a 0.95 sleep efficiency. There are two major outliers. A change in habit of sleep efficiency or a bad sleep efficiency overall can cause some serious health-related side effects, assuming an increased time in bed will correlate to a decreased time active.

### Average Heart Rate
Calculated the average daily heart rate of users:
```
heartrate$DateTime <- as.POSIXct(heartrate$DateTime)
average_heart_rate <- heartrate %>%
  group_by(Id, Date) %>%
  summarise(avg_heart_rate = mean(Value))
```
Plotted it over time for each user: 
```
average_heart_rate$Date <- as.Date(average_heart_rate$Date)
ggplot(average_heart_rate, aes(x = Date, y = avg_heart_rate, color = Id)) +
  geom_line() +
  labs(x = "Date", y = "Average Heart Rate") +
  ggtitle("Average Heart Rate per Date for Each ID") +
  scale_color_discrete(name = "ID")
```
![Screen Shot 2023-06-29 at 2 41 12 PM](https://github.com/JeanneGandon/DataAnalyticsGoogleCertification/assets/138037134/df78f2a0-916d-4815-9f55-c98497f95c79) <br>
Most users remained at a relatively steady Average Heart Rate, but a few had visible major changes in heart rate. This is something that should be addressed when it occurs.

### Total Steps and Activity
Plotted the Total Steps of users compared to the minutes spent in different activity levels (lightly, fairly, very):
```
ggplot(data=activity, aes(x=LightlyActiveMinutes, y=TotalSteps))+geom_jitter(alpha=.5)+
    geom_rug(position="jitter", linewidth=.08)+
    geom_smooth(size =.6)+
    labs(title= "Total Steps vs. Lightly Active minutes")+
    theme_minimal()
ggplot(data=activity, aes(x=FairlyActiveMinutes, y=TotalSteps))+geom_jitter(alpha=.5)+
    geom_rug(position="jitter", linewidth=.08)+
    geom_smooth(size =.6)+
    labs(title= "Total Steps vs. Fairly Active minutes")+
    theme_minimal()
ggplot(data=activity, aes(x=VeryActiveMinutes, y=TotalSteps))+geom_jitter(alpha=.5)+
    geom_rug(position="jitter", linewidth=.08)+
    geom_smooth(size =.6)+
    labs(title= "Total Steps vs. Very Active minutes")+
    theme_minimal()
```
![Screen Shot 2023-06-29 at 3 02 37 PM](https://github.com/JeanneGandon/DataAnalyticsGoogleCertification/assets/138037134/916914f2-b334-4162-8706-20f1983e3623) <br>
![Screen Shot 2023-06-29 at 3 02 58 PM](https://github.com/JeanneGandon/DataAnalyticsGoogleCertification/assets/138037134/2cf41913-6ae0-4918-99d1-3627130ede48) <br>
![Screen Shot 2023-06-29 at 3 03 29 PM](https://github.com/JeanneGandon/DataAnalyticsGoogleCertification/assets/138037134/6b0a0e18-ca75-472a-88fd-499b5f464c32) <br>
These graphs show that lightly active minutes are the most correlated to the total steps. the other two graphs plateau, which shows that fairly and very active work is not related to steps but most likely represents some other form of exercise, likely more static.

## 5. SHARE



## 6. ACT
