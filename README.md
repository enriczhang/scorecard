# scorecard

This R package makes the development of credit risk scorecard efficiently by providing functions such as information value (iv), variable filter (var_filter), optimal woe binning (woebin, woebin_ply, woebin_plot), scorecard scaling (scorecard, scorecard_ply) and performace evaluation (perf_plot, perf_psi).

## Installation

You can install scorecard from github with:

``` r
# install.packages("devtools")
devtools::install_github("shichenxie/scorecard")
```

## Example

This is a basic example which shows you how to develop a common credit risk scorecard:

``` r
# Traditional Credit Scoring Using Logistic Regression
# load germancredit data
data("germancredit")

# rename creditability as y
dt <- data.table(germancredit)[, `:=`(
  y = ifelse(creditability == "bad", 1, 0),
  creditability = NULL
)]

# breaking dt into train and test ------
set.seed(125)
dt <- dt[sample(nrow(dt))]
# rowname of train
set.seed(345)
rn <- sample(nrow(dt), nrow(dt)*0.6)
# train and test dt
dt_train <- dt[rn]; dt_test <- dt[-rn];

# woe binning ------
bins <- woebin(dt_train, "y")

train <- woebin_ply(dt_train, bins)
test <- woebin_ply(dt_test, bins)

# glm ------
m1 <- glm( y ~ ., family = "binomial", data = train)
# summary(m1)

# Select a formula-based model by AIC
m_step <- step(m1, direction="both")
m2 <- eval(m_step$call)
# summary(m2)

# performance ------
# predicted proability
train$pred <- predict(m2, type='response', train)
test$pred <- predict(m2, type='response', test)

# ks & roc plot
perf_plot(train$y, train$pred, title = "train")
perf_plot(train$y, train$pred, title = "test")

# score
card <- scorecard(bins, m2)

# score
train$score <- scorecard_ply(dt_train, card)
test$score <- scorecard_ply(dt_test, card)

# psi
perf_psi(train$y, train$score, test$y, test$score, x_limits = c(0, 700), x_tick_break = 100)

```
