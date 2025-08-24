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
- update_median_census_tracts: Updates the new census tracts with the values from the old census tract for the period provided. This function is only intended for median variables
- update_sum_census_tracts: Updates the values for new census tracts with the values from the old census tract, also applying their ratios from the year 2020 and the old census tract data. This function is only intended for sum variables
- calculate_percentages: Converts category counts to percentages by year
- get_row: Fetches the row of data for the specified census tract ID

The script is easily adaptable to other Census features using the ACS identified and name of variable. Users adapting this notebook to other regions must update State FIPS, County FIPS, and any tract mappings.


## House_Value_Prediction.ipynb

This code trains and evaluates models to predict median house value using data from 2012-2022, then produces forecasts for 2023–2025. It supports linear OLS/Lasso (with LassoCV for alpha selection), XGBoost with Optuna tuning and year-based walk-forward CV, and LSTM and neural network models with KerasTuner. Preprocessing steps include  missing value imputation, log transformation, MinMax scaling, and creating lag features.

The script contains the following functions:

- reshape_wide_to_long: Melts the DataFrame to long format and adds 'Year' column for analysis
- fill_missing_values / fill_missing_values_multiple_fields: Handles NA values using linear interpolation then forward fill/backward fill
- log_transform_column / log_transform_columns: Apply log1p transformation.
- scale_by_column / scale_features: Scales features using MinMacScaler. Scalers are stored for inverse transformation after predictions.

For models utilizing tuning, an additional validation split is added for the tuning process. The "Final Model" section for these models utilizes the optimized hyperparameters found from tuning and trains the model on the whole training set without validation.

The evaluation metrics utilized throughout the model are MAE, RMSE, and R-squared. Models are trained to minimize MSE.



## Data_Mapping.ipynb

This notebook merges CSV datasets with GeoJSON tract data from TIGERWeb API to create tract-level maps for current and predicted housing variables.

After the GeoJSON data for the desired State and County is fetched successfully and stored, a merged GeoJSON file will need to be made with each of the .csv files that you wish to map. After loading the county GeoJSON, set the input CSV and output GeoJSON paths for each variable you wish to map.

The exception is if you're using a prediction dataset, which does not include the Unique_ID column. You will additionally need to type in your State FIPS and County FIPS where it says 34 and 021 in the code block that says "Future Predicted House Value Data" at the top.

Mapping options include:
- Variables over periods of time
- Variables divided into quantiles
- Predicted future values
- Change in value (e.g. change in median house value between years ($) (2012)
- Change in value as a percentage

Be cautious of possible outliers distorting the color scale on maps and making it much more difficult to discern smaller variations in the data. Manually setting vmin and vmax and separately shading values outside that range as separate colors ensures more proper visualization.
