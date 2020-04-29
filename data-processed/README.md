# Data submission instructions

This page is intended to provide teams with all the information they need to
submit forecasts.
Data in this directory should be added to the repository through a pull request
so that automatic data validation checks are run.

These instructions provide detail about the 
[data format](#Data-formatting) as well as 
[validation](#Data-validation)
that you can do prior to this pull request. 
In addition, we describe 
[meta-data](#Meta-data) that each model should provide.

## Data formatting

The automatic check validates both the filename and file contents to ensure the 
file can be used in the visualization and ensemble forecasting.


### Subdirectory

Each subdirectory within this data-processed/ directory has the format

    team-model
    
where 

- `team` is the teamname and 
- `model` is the name of your model. 


### Filenames

Each file within the subdirectory should have the following format

    YYYY-MM-DD-team-model.csv
    
where

- `YYYY` is the 4 digit year, 
- `MM` is the 2 digit month,
- `DD` is the 2 digit day,
- `team` is the teamname, and
- `model` is the name of your model. 

The date YYYY-MM-DD is the date the forecasts were made from your mdoel, 
i.e. the most recent data is YYYY-MM-DD.

The `team` and `model` in this file must match the `team` and `model` in the
directory this file is in.


### File format

The file must be a comma-separated value (csv) file with the following 
columns (in any order):

- `forecast_date`
- `target`
- `target_end_date`
- `location`
- `type`
- `quantile`
- `value`

Additional columns are allowed, but ignored. 

Each row in the file is either a point or quantile forecast for a location on a 
particular date for a particular target. 


#### `forecast_date`

Values in the `forecast_date` column must be a date in the format

    YYYY-MM-DD
    
This is the date the forecasts were made and must match the YYYY-MM-DD in the
filename.


#### `target`

Values in the `target` column must be a character (string) and be one of the 
following specific targets:

- "N day ahead cum death" where N is a number between 1 and 130
- "N day ahead inc death" where N is a number between 1 and 130
- "N wk ahead cum death"  where N is a number between 1 and  20
- "N wk ahead inc death"  where N is a number between 1 and  20
- "N day ahead inc hosp"  where N is a number between 1 and 130

All forecasts will be compared to the 
[daily reports containing death data from the JHU CSSE group](https://github.com/CSSEGISandData/COVID-19/blob/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_deaths_US.csv) 
as the gold standard reference data for deaths in the US. 
Note that there are significant differences 
(especially in daily incident death data) 
between the JHU data and another commonly used source, from the New York Times. 
The team at UTexas-Austin is tracking this issue on [a separate GitHub repository](https://github.com/spencerwoody/covid19-data-comparsion).


For week-ahead forecasts with `forecast_date` of Sunday or Monday of EW12, a 1 week ahead forecast corresponds to EW12 and should have `target_end_date` of the Saturday of EW12. For week-ahead forecasts with `forecast_date` of Tuesday through Saturday of EW12, a 1 week ahead forecast corresponds to EW13 and should have `target_end_date` of the Saturday of EW13. A week-ahead forecast should represent the total number of incident deaths or hospitalizations within a given epiweek (from Sunday through Saturday, inclusive) or the cumulative number of deaths reported on the Saturday of a given epiweek. We have created [a csv file](../template/covid19-death-forecast-dates.csv) describing forecast collection dates and dates for which forecasts refer to can be found.


##### N day ahead cum death

This target is the cumulative number of deaths predicted by the model for 
N days after `forecast_date`. 

##### N day ahead inc death

This target is the incident (daily) number of deaths predicted by the model
on day N after `forecast_date`. 

##### N wk ahead cum death

This target is the cumulative number of deaths predicted by the model up to 
and including N weeks after `forecast_date`. 

##### N wk ahead inc death

This target is the incident (weekly) number of deaths predicted by the model 
during the week that is N weeks after `forecast_date`. 

##### N day ahead inc hosp

This target is the incident (weekly) number of deaths predicted by the model
on day # after `forecast_date`.



#### `target_end_date`

Values in the `target_end_date` column must be a date in the format

    YYYY-MM-DD
    
This is the date for the forecast `target`. 
For "# day" targets, `target_end_date` will be # days after `forecast_date`.
For "# wk" targets, `target_end_date` will be the Saturday at the end of the 
week time period.


#### `location`

Values in the `location` column must be

- "US" or
- a two-digit number representing the US state, territory, or district 
[fips numeric code](../template/state_fips_codes.csv). 

This location identifies the geographical location for the forecast.


#### `type`

Values in the `type` column are either

- "point" or
- "quantile". 

This value indicates whether that row corresponds to a point forecast or a 
quantile forecast. 
Point forecasts are used in visualization while quantile forecasts are used in
visualization and in ensemble construction.


#### `quantile`

Values in the `quantile` column are either missing (if `type` is "point") or 
a quantile in the format

    0.###
    
For quantile forecasts, this value indicates the quantile for the `value`
in this row. 

Teams should provide the following quantiles:


```r
c(0.01, 0.025, seq(0.05, 0.95, by = 0.05), 0.975, 0.99)
```

```
##  [1] 0.010 0.025 0.050 0.100 0.150 0.200 0.250 0.300 0.350 0.400 0.450
## [12] 0.500 0.550 0.600 0.650 0.700 0.750 0.800 0.850 0.900 0.950 0.975
## [23] 0.990
```


#### `value`

Values in the `value` column are numeric indicating the "point" or "quantile"
prediction for this row. 
For a "point" prediction, `value` is simply the value of that point prediction
for the `target` and `location` associated with that row. 
For a "quantile" prediction, `value` is the inverse of the cumulative 
distribution function (CDF)
for the `target`, `location`, and `quantile` associated with that row.

An example inverse CDF is below.


```r
mu = log(50000)
q = 0.800
value = qlnorm(q, mu)
curve(qlnorm(x, mu), 0.1, 0.9,
      xlab = "quantile", 
      ylab = "inverse CDF",
      main = paste("Example `value` at quantile", q))
segments(q, 0, q, value)
segments(0, value, q, value)
axis(2, at = value, "value", las = 2)
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2-1.png)



## Data validation

To ensure proper data formatting, 
pull requests for new data in data-processed/ will be automatically run.



### Pull request data validation

When a pull request is submitted, 
the data are validated through [Travis CI](https://travis-ci.org/) which runs
the tests in [test-formatting.py](../code/validation/test-formatting.py).
The intent for these tests are to validate the requirements above and 
specifically enumerated [on the wiki](https://github.com/reichlab/covid19-forecast-hub/wiki/Validation-Checks).
Please [let us know](https://github.com/reichlab/covid19-forecast-hub/issues) 
if the wiki is inaccurate.

If the pull request fails, please 
[follow these instructions](https://github.com/reichlab/covid19-forecast-hub/wiki/Troubleshooting-Pull-Requests)
for details on how to troubleshoot.

For those familiar with python, 
you can run the tests in 
[test-formatting.py](../code/validation/test-formatting.py) 
prior to submitting a pull request to ensure the data will validate. 


### R data validation

For those familiar with R (but not python),
there is a separate set of tests that may be useful to diagnose data 
formatting issues in
[perform_plausibility_checks.script.R](../code/perform_plausibility_checks.script.R).
We intend to keep these tests in sync with the python checks 
automatically run during a pull request.
If you discover any discrepancies,
please [let us know](https://github.com/reichlab/covid19-forecast-hub/issues).



## Meta-data

Participating teams must provide a metadata file (see [example](../data-processed/UMass-ExpertCrowd/metadata-UMass-ExpertCrowd.txt)), including methodological detail about their approach and a link to a file 
(or a file itself) describing the methods used. 