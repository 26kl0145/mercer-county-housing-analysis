# mercer-county-housing-analysis

This repository contains code and analysis for exploring housing price trends in Mercer County, New Jersey, from 2012 to 2022. Using U.S. Census data, machine learning models, and geospatial mapping, this project investigates socioeconomic disparities in housing values and predicts future housing prices.

The research highlights persistent economic inequality across the county, particularly between Trenton and Princeton, and demonstrates the utility of deep learning models for forecasting housing market trends.

## Key Features
- Extract and process Census Bureau data for median house values, household income, racial composition, and educational attainment across Mercer County census tracts.
- Train and evaluate multiple predictive models for housing prices, including:
  - Linear OLS and Lasso regression
  - XGBoost with Optuna tuning
  - Neural networks and LSTM models via KerasTuner
- Forecast median house values for 2023–2025 using the optimized models.
- Visualize current and predicted data on tract-level maps using GeoJSON and the TIGERWeb API.

## Census_Data_Extraction.ipynb

This code accesses the Census dataset and creates CSV files with median household income, median house value, racial demographics (counts and percentages), and educational attainment (counts and percentages) for all census tracts in the New Jersey Mercer County area.

U.S. Census Bureau geographic boundaries of census tracts are revised every 10 years, with tracts often being split into two tracts in order to better reflect drastic changes in population or income distribution in the area. Consequently, the code estimates values for newly defined tracts by copying median statistics and proportionally allocating sum statistics from prior tract boundaries.

The script contains the following functions:

- fetch_census_data_by_field: Fetches ACS data given the FIPS codes for the region and the Census code for the field
- prepare_yearly_data: Adds Unique_ID and keeps year-specific columns for a field
- update_median_census_tracts: Updates the new census tracts with the values from the old census tract for the period provided. This function is only intended for median characteristics
- update_sum_census_tracts: Updates the values for new census tracts with the values from the old census tract, also applying their ratios from the year 2020 and the old census tract data. This function is only intended for sum characteristics
- calculate_percentages: Converts category counts to percentages by year
- get_row: Fetches the row of data for the specified census tract ID

The script is easily adaptable to other features. In order to utilize the code and generate DataFrames for desired characteristics, the user must change the State FIPS and County FIPS to the location of interest, and the field and field name must be changed to the ACS identifier and the name of the characteristic.

The old census tracts and their corresponding new census tracts must also be changed. I could not find a simple way to acquire which census tracts were divided in 2020. To identify splits, I opened up the map of census tracts for the county in 2020 (found here: https://www.census.gov/geographies/reference-maps/2020/geo/2020pl-maps/2020-census-tract.html) and comparing it to the map of census tracts for the county in 2010 (found here: https://www.census.gov/geographies/reference-maps/2010/geo/2010-census-tract-maps.html)

Additionally, I recommend looking at the dataset over time before using the update functions. Any rows which only contain values before 2020 were removed in the revision of census tracts in 2020. Any rows which only contain values for and after 2020 were the result of this revision.



## House_Value_Prediction.ipynb

This code uses the extracted CSVs (2012–2022) to train and compare models for predicting median house value, then produces forecasts for 2023–2025. It supports linear OLS/Lasso (with LassoCV for alpha selection), XGBoost with Optuna tuning and year-based walk-forward CV, and LSTM and neural network models with KerasTuner. Preprocessing steps include filling missing values, log transformation, MinMax scaling, and creating lag features.

The script contains the following functions:

- reshape_wide_to_long: Melts the DataFrame to long format and adds 'Year' column for more simple analysis
- fill_missing_values / fill_missing_values_multiple_fields: All NA values in the DataFrame are filled using linear interpolation then forward fill/backward fill
- log_transform_column / log_transform_columns: Apply log1p transformation to target/features.
- scale_by_census_tract: scales a feature of the DataFrame using MinMaxScaler between 0-1. The scalers are stored so that they may be reverse normalized after predictions
- scale_by_column / scale_features: Scales features using MinMacScaler. Scalers are stored for inverse transformation after predictions.

For models utilizing tuning, an additional validation split is added for the tuning process. The "Final Model" section for these models utilizes the optimized hyperparameters found from tuning and trains the model on the whole training set without validation.

The evaluation metrics utilized throughout the model are MAE, RMSE, and R-squared. Models are trained to minimize MSE.



## Data_Mapping.ipynb

This code merges the CSV dataset with GeoJSON data to produce tract-level maps for current and predicted characteristics. GeoJSON data is fetched from TIGERWeb API.

This script has no functions.

After the GeoJSON data for the desired State and County is fetched successfully and stored, a merged GeoJSON file will need to be made with each of the .csv files that you wish to map. After loading the county GeoJSON, set the input CSV and output GeoJSON paths for each variable you wish to map.

The exception is if you're using a prediction dataset, which does not include the Unique_ID column. You will additionally need to type in your State FIPS and County FIPS where it says 34 and 021 in the code block that says "Future Predicted House Value Data" at the top.

After this, maps of the data are constructed in various forms.

- Characteristics over periods of time
- Characteristics divided into quantiles
- Predicted characteristics
- Change in value (e.g. change in median house value between years ($) (2012)
- Change in value as a percentage

For maps of changes in value between years, my maps have red sections signifying increases in house value, while blue sections signify decreases in house value, centered around 0. Be cautious of possible outliers, as one extremely high increase or decrease can disproportionately affect the color scale and make it much more difficult to discern smaller variations in the data. The work-around used in the final code block of the script was to set the vmin and vmax myself, shading any values outside that range as separate colors (representing those outliers).
