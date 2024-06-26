# Project-Tesla-Stock-Price-Prediction
In this project, Exploratory Data Analysis is carried out on TESLA Stock. Here, EDA is performed with different data analysis technologies viz. R  is used to predict the stock prices for January and also to predict percentage increases or decreases in the stock prices.
![image](https://github.com/LangatErick/Project-Tesla-Stock-Price-Prediction/assets/124883947/801a8c48-0f7f-4e83-98db-323877b246b7)

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Tesla Stock Price Prediction

In this project, **Exploratory Data Analysis** is carried out on TESLA Stock data obtained from Yahoo Finance:

Here, EDA has performed using R,  which is used to predict the stock prices for January and also predict the percentage increase or decrease in the stock prices.

```{r warning=FALSE, message=FALSE}
#import libraries
library(tidyverse)
library(ggplot2)
library(zoo)

```
## Part 1: Data Retrieval and Processing.
First and foremost, data is collected from the above-mentioned source for the period of 1st January 2021 to 1st January 2022. The gathered data is then pre-processed and cleaned by removing empty rows and duplicates.
```{r warning=FALSE, message=FALSE}
#import dataset
TSLA <- read_csv("TSLA.csv")
sum(duplicated(TSLA$Date))
colSums(is.na(TSLA))
glimpse(TSLA)
```

```{r}
#manipulate date variable
TSLA$Date <- ymd(TSLA$Date)
```
## Part 2: Exploratory Data Analysis (EDA)
The second part of this project is Exploratory Data Analysis (EDA), which is performed in 4 different ways.

- Assessing basic stock performance: In this first step, closing prices of TESLA stocks are simply plotted against the timeframe to understand the basic movements of stock prices.
- Trend Analysis using Simple Moving Average (SMA): Simple Moving Averages of stocks are carried out over 30 days, 100 days, and 200 days intervals. SMAs help understand stock movements at different time intervals. For example, identifying short-term or long-term trends for stocks.
- Seasonal Trend Analysis: Here, stocks are analyzed based on the monthly average of their closing prices. This can help in understanding which seasons are best for investing in a particular stock and which can be the best time to sell a stock.
- Volatility Analysis: Volatility Analysis is carried out to understand how frequently stock prices can vary. This helps in understanding the risks involved in investing in a stock and also it shows the tendency of the stock to diverge from its flow frequently.
### Basic trend Analysis using Closing prices of stock data

```{r warning=FALSE, message=FALSE}
#visualize
theme_set(theme_test())
TSLA %>% 
    ggplot(aes(x=Date))+
    geom_line(aes(y=Close), col='deepskyblue')+
    scale_x_date()+
    scale_x_date(date_breaks = "2 months",date_labels=("%b %y"))+
    theme(axis.text.x = element_text(
      angle = 45,
      hjust = 1
    )) +
  theme(panel.grid.major = element_blank(),
    panel.grid.minor = element_blank()) +
  labs(
    title = "Tesla Stock Price Trend Analysis",
    x = "Date",
    y = "Stock Price (USD)"
  )

```
![image](https://github.com/LangatErick/Project-Tesla-Stock-Price-Prediction/assets/124883947/94e4699d-9c5a-4c06-9d46-ad2133a1b273)

### Find Simple Moving Averages for 30, 100 and 200 days
```{r warning=FALSE, message=FALSE}
TSLA1 <- TSLA %>%
  mutate(SMA_30 = rollmean(Close, k = 30, fill = NA),
         SMA_100 = rollmean(Close, k = 100, fill = NA),
         SMA_200 = rollmean(Close, k = 200, fill = NA))
```

### Plot moving averages along with closing prices

```{r}
# Create breaks and labels for x-axis
tesla_data_long <- TSLA1 %>%
  pivot_longer(cols = c(Close, SMA_30, SMA_100, SMA_200), names_to = "Series", values_to = "Value")
```

```{r warning=FALSE, message=FALSE}
# Plot the reshaped data
ggplot(data = tesla_data_long, aes(x = Date, y = Value, color = Series)) +
  geom_line() +
  labs(title = "Tesla Stock Price with Moving Averages (30, 100, and 200 days)",
       x = "Date",
       y = "Stock Price (USD)",
       color = "Series") +
  scale_color_manual(values = c("Close" = "blue", 
                                 "SMA_30" = "red", 
                                 "SMA_100" = "green", 
                                 "SMA_200" = "orange"),
                     labels = c("Close", "SMA 30", "SMA 100", "SMA 200")) +
  scale_x_date(date_breaks = "2 months", date_labels =("%b %y"))+
  theme_bw()+
  theme(panel.grid.major = element_blank(),
    panel.grid.minor = element_blank(),
    legend.position = "bottom"
)+
  theme(axis.text.x = element_text(
    angle = 45,
    hjust = 1
  ))
```
![image](https://github.com/LangatErick/Project-Tesla-Stock-Price-Prediction/assets/124883947/7c98f701-0e2d-428a-80fe-f28360c72a06)

### Calculate rolling standard deviation for 100 day and perform Volatility Analysis

```{r warning=FALSE, message=FALSE}
tesla_data <- TSLA1 %>%
  mutate(Volatility_100 = rollapply(Close, width = 100, 
                                    FUN = sd, 
                                    fill = NA))

ggplot(data = tesla_data, aes(x = Date)) +
  geom_line(aes(y = Volatility_100, 
                color = "100-Day"), 
            linetype = "solid") +
  labs(title = "Tesla Stock Volatility Analysis (100 days)",
       x = "Date",
       y = "Volatility",
       color = NULL) +
  scale_x_date(date_labels=("%b %y"), breaks="2 months")+
  theme_bw()+
  theme(axis.text.x = element_text(
    angle = 45,
    hjust = 1
  ))+
  theme(panel.grid.major = element_blank(),
    panel.grid.minor = element_blank(),
    legend.position="none"
)
```
![image](https://github.com/LangatErick/Project-Tesla-Stock-Price-Prediction/assets/124883947/4626661e-9b35-4d59-a875-ef95a4543e4f)

### Perform Seasonal Trend Analysis by calculating monthly average prices.

```{r warning=FALSE, message=FALSE}
# Create 'Month' column from 'Date' column
tesla_data <- TSLA1 %>%
  mutate(Month = format(Date, "%Y-%m"))

# Calculate the average stock price for each month
monthly_avg <- tesla_data %>%
  group_by(Month) %>%
  summarize(Avg_Price = mean(Close, na.rm = TRUE))

# Modify the 'Month' column format to display in 'Year-Month' format (YYYY-MM)
monthly_avg$Month <- as.yearmon(monthly_avg$Month)

# Plot seasonal trend using scatter plot
ggplot(monthly_avg, aes(x = Month, y = Avg_Price)) +
  geom_point(color = "blue", size = 3) +
  geom_smooth(method = "lm", se = FALSE, color = "red", linetype = "dashed") +  # Add a linear trendline
  labs(title = "Seasonal Trend of Tesla Stock Price by Month",
       x = "Month",
       y = "Average Stock Price (USD)") +
  theme_bw()+
  theme(panel.grid.major = element_blank(),
    panel.grid.minor = element_blank()
)
```
![image](https://github.com/LangatErick/Project-Tesla-Stock-Price-Prediction/assets/124883947/ec416eb3-af6f-49dc-a7a3-04ff271d87f3)
