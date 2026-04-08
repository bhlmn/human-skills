# Train Test Split

A list of best practices for this crucial step in machine learning development.

## Good split breakdown

The conventional wisdom is 80% of data for training, 20% for testing, but there is more to this. Distinct sets within a split include:

* Training – data the model will use to learn signals.
* Validation – data the model will use to calculate validation loss, support early stopping, and compare models (when hyperparameter tuning)
* Test – data to estimate the predictive performance on unseen data

## Creating splits

Since I am using SQL a lot these days (specifically BigQuery), I can use [`farm_fingerprint`](https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/hash_functions#farm_fingerprint) to hash a string identifier into a signed integer, then use the modulus to split the data:

``` sql
with

hashed as (

    select
        *, 
        mod(
            abs(
                farm_fingerprint(unique_string_key)
            ), 
            10
        ) as hashed_assignment
    
    from feature_set

), 

split as (

    select
        * except (hashed_assignment), 

        case
            when hashed_assignment <= 7 then 'Training'
            when hashed_assignment = 8 then 'Validation'
            when hashed_assignment = 9 then 'Test'
        end as split_category
    
    from hashed

)

select * from split
```

## Validating split feature and label distributions

ML models are most predictive when the training data represents unseen data well. They way we best test for this is ensuring that, after we've split the data, the training, validation, and test sets have similar feature and target distributions.

Questions to ask and confirm in code:
* After the split, does observation volume in each split make sense? (i.e., did we get an 80/10/10 split?)
* Does the target have similar distributions across the training, validation, and test sets?
* Do features have similar distributions across the training, validation, and test sets?
* Have outliers (observations that represent more noise than signal) been removed from each set?
* Are no instances of the test set found in the training set?