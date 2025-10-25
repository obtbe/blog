---
title: Linear Regression vs Logistic Regression - Understanding the Differences
date: 2024-12-15 12:00:00
tags: [statistics, regression, analysis]
---


Data analysis often uses two tools: linear regression and logistic regression. They sound alike, but they answer different questions. This article breaks down their differences with clear examples and the math behind them, so you can choose the right one for your data.

## Linear Regression: Predicting Numbers

Linear regression predicts numbers by finding a straight-line relationship between inputs and an output. Picture yourself running a lemonade stand, trying to predict daily sales based on the day’s temperature. Hotter days might mean more customers, and linear regression can model this.

Imagine you track 40 days, recording temperature in degrees Fahrenheit and lemonade cups sold. On a 65°F day, you sell 120 cups, but on an 85°F day, you sell 160 cups. Linear regression creates a line that fits this pattern. The equation is:

```shell
Cups Sold = b0 + b1 * Temperature
```

Here, b0 is the starting point (cups sold if temperature were 0°F), and b1 is the slope (extra cups per degree). Your data might give:

```shell
Cups Sold = 30 + 1.5 * Temperature
```

For a 70°F day, you’d predict 30 + 1.5 * 70 = 135 cups. The math minimizes the squared differences between actual sales and the line’s predictions, called least squares. For a 65°F day with 120 cups sold, the prediction is 30 + 1.5 * 65 = 127.5 cups, so the error is 120 - 127.5 = -7.5. Square it (7.5^2 = 56.25), sum all errors across days, and adjust b0 and b1 to make the sum as small as possible.

Another example: predicting a runner’s 5K time based on weekly training hours. Data from 60 runners shows 10 hours per week gives a 25-minute finish, while 15 hours gives 22 minutes. The model might be:

```shell
Finish Time = 30 - 0.5 * Training Hours
```

A runner training 12 hours would finish in 30 - 0.5 * 12 = 24 minutes. Linear regression works for number predictions, assuming a straight-line relationship and evenly spread errors.

## Logistic Regression: Predicting Yes or No

Logistic regression predicts whether something happens, like if a moviegoer buys popcorn. It gives a probability between 0 and 1, which you can turn into yes or no. The model uses a logistic function to curve predictions into this range.

Suppose a theater tracks 150 customers, noting their age and whether they bought popcorn. A 25-year-old has a 40% chance of buying, while a 50-year-old has a 75% chance. The logistic regression equation is:

```shell
P(Buy Popcorn) = 1 / (1 + e^(-(-3.5 + 0.08 * Age)))
```

Here, P(Buy Popcorn) is the probability, b0 is the starting point, and b1 adjusts how age affects the chance. Your data might give:

```shell
P(Buy Popcorn) = 1 / (1 + e^(-(-3.5 + 0.08 * Age)))
```

For a 30-year-old, calculate -3.5 + 0.08 * 30 = -1.1, so e^(-(-1.1)) = e^1.1 ≈ 3.004. Then, P(Buy Popcorn) = 1 / (1 + 3.004) ≈ 0.25, or a 25% chance. If the probability is above 0.5, predict “yes”; otherwise, “no”. The math adjusts b0 and b1 to maximize the chance the model matches actual yes-or-no choices.

Another example: predicting if a plant survives based on weekly watering days. Data from 80 plants shows 2 days of watering gives a 30% survival chance, while 5 days gives 80%. The model might be:

```shell
P(Survive) = 1 / (1 + e^(-(-4 + 0.9 * Watering Days)))
```

A plant watered 3 days has a probability of 1 / (1 + e^(-(-4 + 0.9 * 3))) ≈ 0.43, or 43%, so predict “no”. Logistic regression fits yes-or-no outcomes like these.

## How They Differ

Linear and logistic regression tackle different tasks. Here’s a breakdown:

- **Output**: Linear regression predicts numbers, like lemonade sales or 5K times. Logistic regression predicts probabilities for yes-or-no events, like buying popcorn or plant survival.
- **Inputs**: Both use numbers or categories (e.g., temperature or weekday/weekend), but logistic regression often prefers simpler inputs for clear probability curves.
- **Math**: Linear regression assumes a straight line and minimizes squared errors, calculated as:

```shell
Error = Sum (Actual - Predicted)^2
```

- Logistic regression assumes a curved relationship and maximizes the likelihood of correct yes-or-no predictions.
- **Coefficients**: In linear regression, b1 shows the output change per input unit (e.g., 1.5 cups per degree). In logistic regression, b1 shifts the probability curve, affecting the odds of “yes”.

## When to Use Each

Use linear regression for number predictions. For example, a bakery might predict muffin sales based on the day’s foot traffic. Data from 50 days shows 200 visitors yield 80 muffins sold, while 300 visitors yield 110. The model could be:

```shell
Muffins Sold = 20 + 0.3 * Visitors
```

For 250 visitors, predict 20 + 0.3 * 250 = 95 muffins. Or use it to estimate a house’s energy bill based on its size, like a 1,500-square-foot house costing $120 monthly.

Use logistic regression for yes-or-no predictions. A pet store might predict if a customer adopts a puppy based on visit time. Data from 100 visits shows 10-minute visits have a 20% adoption chance, while 30-minute visits have a 60% chance. Or a teacher might predict if a student submits homework based on attendance, with 90% attendance giving an 85% chance.

Can you use them interchangeably? Not really. Linear regression on yes-or-no data might predict nonsense, like a 1.4 probability. Logistic regression on numbers doesn’t fit since it’s built for binary outcomes. Match the model to your question: numbers for linear, yes-or-no for logistic.

Both models are powerful for data analysis. Linear regression handles number predictions, and logistic regression solves yes-or-no problems. Pick the right one for your data’s question, and you’ll get clear, useful answers.