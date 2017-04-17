# PayTM Weblog Challenge 

## Execution
Execute via Spark 1.6.0 spark-shell
 
## Implementation
The solution was done via Spark RDD APIs. I will preface the code with I am new to Spark and Scala. With more time and farmiliarity with spark / sbt / spark-testing frameworks I would improve the code by instead of just being a spark-shell script I would create a deployable application and associated unit tests.

## Inputs
The solution assumes that sample.log.gz is stored in root HDFS folder

## Outputs
The solution publishes to root HDFS folder a single part file for each of:
1. unique_session_hits (911728 unique session hits)
2. most_engaged_users_with_session (90544 users)

I use coalesce to get 1 part. The output parts are included in output directory of project as part files and csv.

## Sessionization
The sessionization is performed via the default proposed 15 minute window. 
 
## Optimal bin window analysis notes
An approach to identify the optimal window could look at the distribution of intra-visit durations and choosing a thresolhold T such that X% of clicks happen within T seconds. For simplicity, if we assume that for any T the number of sessions of a single click are neglible this is ok. To be more thorough it would be important to run through potential values for T, sessionize, and determine if the number of single click sessions  provides appropriate X%.

## MLE Questions notes for potential solutions

These are some basic statistical approaches to make a crude guess at prediction. 

#### 1. Predict the expected load (requests/second) in the next minute

#### Average
Use the average requests/second. This is very crude - there is probably some dependency minute to minute for # of requests

#### Regression
Bucket the requests into second buckets and run a sliding 2 second window over it to get a 2D dataset that shows (r_t1, r_t2) for t1 = 1 ... T - 1, t2 = 2 ... T where r_t1 is the number of request in first minute and r_t2 is number of requests in second minute. If a scatter plot reveals some sort of algebraic relationship, we could try a regression model.

### 2. Predict the session length for a given IP

#### Average 1
Use the average session length 

#### Average 2
Consider the page landed on, use the average session length across sessions containing that page or fallback to Average 1

#### Average 3
Consider the page landed on, use the average remaining session length across sessions containing that page or fallback to Average 2 or Average 1

#### Known IP Option if have "enough" history
If have enough info about visitor then make guess from their behavior using one of the above approaches

### 3. Predict the number of unique URL visits by a given IP

#### Average 1
Use the average unique URL visits

#### Average 2
Consider the page landed on, use the average unique URL visists sessions containing that page or fallback to Average 1

#### Average 3
Consider the page landed on, use the average remaining unique URL visits across sessions containing that page or fallback to Average 2 or Average 1

#### Known IP Option if have "enough" history
If have enough info about visitor then make guess from their behavior using one of the above approaches
