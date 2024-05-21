Forecasting
===========

.. note:: 

   First, we want to find the ideal parameters for "initial", "period" and "horizon" using cross validation. This is done on the basis of all zones. These are then fixed for all further steps, as a calculation for each zone individually would be too time-consuming.

   The hyperparameters are then tuned using cross validation, but now once for the whole of Manhattan and once for each zone.

   The recommendations for cross validation and hyperparameter settings in the Prophet documentation were always followed by the official recommendation in the documentation: https://facebook.github.io/prophet/docs/diagnostics.html.

   Here we only consider the highlight. For a detailed insight into the complete procedure, see:

   https://github.com/Hakan1998/NYC-Taxi-Demand-Forecast/blob/main/Projektseminar__Forecast_Model_.ipynb

Function: Find Best Cross-Validation Parameters
------------------------------------------------

.. function:: find_best_cv_params(train_data)

   Find the best parameters for cross validation using Prophet.

   :param train_data: A pandas DataFrame containing the training data with a 'time_bin' column and a 'demand' column.
   :type train_data: pandas.DataFrame
   :return: The best cross-validation parameters.
   :rtype: dict

   This function prepares the training data by aggregating demand values over time bins. It then fits a Prophet model to the aggregated data. Next, it defines different combinations of cross-validation parameters for 'initial', 'period', and 'horizon'. For each parameter combination, it performs cross-validation using Prophet's built-in function `cross_validation` and calculates the mean absolute percentage error (MAPE) using `performance_metrics`. The function returns the cross-validation parameters with the lowest MAPE.

.. code-block:: python

      import pandas as pd
      from fbprophet import Prophet

      def find_best_cv_params(train_data):
          # Prepare the data
          df = train_data.copy().groupby('time_bin')['demand'].sum().reset_index().rename(columns={'time_bin': 'ds', 'demand': 'y'})
          
          # Fit Prophet model
          model = Prophet().fit(df)
          
          # Define all cross-validation parameter combinations
          all_cv_params = [
              {"initial": "7 days", "period": f" {i} days", "horizon": "21 days"} for i in [0.5, 1, 2, 3, 7]
          ] + [
              {"initial": "14 days", "period": f" {i} days", "horizon": "14 days"} for i in [0.5, 1, 2, 3, 7]
          ] + [
              {"initial": "21 days", "period": f" {i} days", "horizon": "7 days"} for i in [0.5, 1, 2, 3, 7]
          ]
          
          # Perform cross-validation for each parameter combination and calculate MAPE
          rmses_cv = []
          for cv_params in all_cv_params:
              df_cv = cross_validation(model, initial=cv_params['initial'], period=cv_params['period'], horizon=cv_params['horizon'])
              df_p = performance_metrics(df_cv)
              rmses_cv.append(df_p['mape'].values[0])
          
          # Create DataFrame to store results
          cv_results = pd.DataFrame(all_cv_params)
          cv_results['mape'] = rmses_cv
          
          # Find the best cross-validation parameters
          best_cv_params = cv_results.loc[cv_results['mape'].idxmin()]
          
          return best_cv_params

With these Parameter we can start to make the Forecast and tune it:

Function: Make Forecast
-----------------------

.. code-block:: python

   def make_forecast(model, df):
       """
       Make a forecast using the given model and DataFrame.

       :param model: The Prophet model to use for forecasting.
       :type model: Prophet
       :param df: The DataFrame containing the data for forecasting.
       :type df: pandas.DataFrame
       :return: The forecasted values.
       :rtype: pandas.DataFrame
       """
       future = model.make_future_dataframe(periods=28 * 24, freq="h", include_history=False)
       forecast = model.predict(future)
       return forecast

This function takes a trained Prophet model and a DataFrame containing the data for forecasting. It generates future time points using the model's `make_future_dataframe` method and makes predictions using the `predict` method. The forecasted values are returned as a DataFrame.


Function: Tune Prophet Parameters
----------------------------------

.. code-block:: python

   def tune_prophet_parameters(df, param_grid):
       """
       Tune Prophet parameters and return the best parameters.

       :param df: A pandas DataFrame containing the training data.
       :type df: pandas.DataFrame
       :param param_grid: A dictionary containing the parameter grid for tuning.
       :type param_grid: dict
       :return: The best-tuned parameters.
       :rtype: dict
       """
       errors = []

       all_params = [dict(zip(param_grid.keys(), v)) for v in itertools.product(*param_grid.values())]

       for params in all_params:
           model = Prophet(**params).fit(df)
           df_cv = cross_validation(model, initial=INITIAL_PERIOD, period=PERIOD, horizon=HORIZON)
           df_p = performance_metrics(df_cv)

           # Check if 'mape' is present, otherwise, use MSE as an alternative metric
           if 'mape' in df_p.columns:
               errors.append(df_p['mape'].values[0])
           else:
               # Use MSE as an alternative metric
               errors.append(df_p['mse'].values[0])

       tuning_results = pd.DataFrame(all_params)
       tuning_results['error'] = errors
       best_params = tuning_results.loc[tuning_results['error'].idxmin()].drop('error')
       return best_params

This function tunes Prophet parameters using cross validation. It iterates through all parameter combinations specified in `param_grid`, fits Prophet models with these parameters to the training data, and evaluates their performance using cross validation. The function returns the parameters with the lowest mean absolute percentage error (MAPE).


Function: Calculate MAPE
-------------------------

.. code-block:: python

   def calculate_mape(table, forecast_column, actual_column):
       """
       Calculate Mean Absolute Percentage Error (MAPE).

       :param table: The DataFrame containing forecast and actual values.
       :type table: pandas.DataFrame
       :param forecast_column: The name of the column containing forecasted values.
       :type forecast_column: str
       :param actual_column: The name of the column containing actual values.
       :type actual_column: str
       :return: The calculated MAPE.
       :rtype: float
       """
       mape = np.mean(np.abs((table[actual_column] - table[forecast_column]) / table[actual_column])) * 100
       return mape

This function calculates the Mean Absolute Percentage Error (MAPE) between forecasted and actual values. It takes a DataFrame `table` containing both forecasted and actual values, as well as the names of the forecasted and actual columns. It returns the calculated MAPE as a float.

Function: Generate cominbed forecast Table
------------------------------------------------

.. function:: generate_combined_forecast_table(train_data, test_data, param_grid)

   Generate a combined forecast table for multiple zones, including both basic and tuned forecasts.

   :param train_data: The training data containing historical demand information.
   :type train_data: pandas.DataFrame
   :param test_data: The test data containing demand information for evaluation.
   :type test_data: pandas.DataFrame
   :param param_grid: A dictionary containing the parameter grid for tuning Prophet models.
   :type param_grid: dict
   :return: A combined forecast table for all zones with both basic and tuned forecasts.
   :rtype: pandas.DataFrame

   This function generates a combined forecast table for multiple zones, including both basic and tuned forecasts. It iterates through each unique zone in the training data, tunes Prophet parameters, fits Prophet models, makes forecasts, and combines the results into a single DataFrame.

   - The function first extracts unique zones from the training data.
   - It then iterates through each zone, tuning Prophet parameters and fitting models using the training data for that zone.
   - For each zone, it generates two forecasts: one with tuned parameters and another with default parameters.
   - The function combines forecasted demand, actual demand (from test data), and zone information into a DataFrame for each zone.
   - Finally, it concatenates the individual zone tables into a single combined forecast table.

   This combined forecast table provides a comprehensive view of both basic and tuned forecasts, facilitating evaluation and comparison of forecasting models.

   .. note:: This function utilizes the `tune_prophet_parameters`, `make_forecast`, and `calculate_mape` functions internally for tuning parameters, generating forecasts, and calculating forecast accuracy metrics respectively.

.. code-block:: python

   def generate_combined_forecast_table(train_data, test_data, param_grid):
       unique_zones = train_data['zone'].unique()
       combined_forecast_table = pd.DataFrame()

       for zone in tqdm(unique_zones, desc='Processing zones'):
           zone_data = train_data[train_data['zone'] == zone].copy().groupby('time_bin')['demand'].sum().reset_index().rename(columns={'time_bin': 'ds', 'demand': 'y'})

           best_params = tune_prophet_parameters(zone_data, param_grid)
           best_model = Prophet(**best_params).fit(zone_data)
           forecast_best = make_forecast(best_model, zone_data)

           default_model = Prophet().fit(zone_data)
           forecast_default = make_forecast(default_model, zone_data)

           table_zone = pd.DataFrame({
               'ds': forecast_best['ds'],
               'forecast_best': round(forecast_best['yhat']).astype(int),
               'forecast_default': round(forecast_default['yhat']).astype(int),
               'Zone': zone
           })

           filtered_test_data_zone = test_data[test_data['zone'] == zone].copy().groupby('time_bin')['demand'].sum().reset_index().rename(columns={'time_bin': 'ds', 'demand': 'test_demand'})
           table_zone = pd.merge(table_zone, filtered_test_data_zone[['ds', 'test_demand']], on='ds', how='left')

           combined_forecast_table = pd.concat([combined_forecast_table, table_zone], ignore_index=True)

       return combined_forecast_table
