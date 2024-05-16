.. _forecast:

==============
Data Preprocessing
==============

.. _process_taxi_data:

process_taxi_data Function
===========================

.. function:: process_taxi_data(data_train, shapefile_path)

   This function processes taxi data by performing spatial operations with zone shapefiles.

   :param data_train: A pandas DataFrame containing taxi data.
   :type data_train: pandas.DataFrame
   :param shapefile_path: The file path to the shapefile containing zone information.
   :type shapefile_path: str
   :return: A pandas DataFrame with processed taxi data including zone and borough information.
   :rtype: pandas.DataFrame

To match each pickup Coordinates with the specific zone of Manhatten:

This function loads the zone shapefile specified by ``shapefile_path``, transforms it to EPSG:4326 coordinate system for consistent comparison, and performs spatial operations with the taxi data provided in the DataFrame ``data_train``. It extracts relevant columns such as "Trip_Pickup_DateTime", "pickup_day", "pickup_hour", "Start_Lon", "Start_Lat", "geometry", "zone", and "borough". The resulting DataFrame includes these columns along with zone and borough information merged from the shapefile. The function returns this processed DataFrame.

   Example Usage::

      import geopandas as gpd
      import pandas as pd

      def process_taxi_data(data_train, shapefile_path):
          gdf_zones = gpd.read_file(shapefile_path).to_crs('EPSG:4326')
          gdf_taxi = gpd.GeoDataFrame(data_train, geometry=gpd.points_from_xy(data_train['Start_Lon'], data_train['Start_Lat']))
          gdf_taxi.crs = "EPSG:4326"
          taxi_with_zones = gpd.sjoin(gdf_taxi, gdf_zones, how='left', op='within')
          result_df = pd.merge(taxi_with_zones[["Trip_Pickup_DateTime", "pickup_day", "pickup_hour", "Start_Lon", "Start_Lat", "geometry", "zone", "borough"]].rename(columns={'geometry': 'geo_point'}),
                                   gdf_zones[['zone', 'borough', 'geometry']], on=['zone', 'borough'], how='left')
          return result_df

      # Example usage
      data = ...  # Load taxi data
      shapefile_path = "/path/to/shapefile.shp"
      processed_data = process_taxi_data(data, shapefile_path)

.. _one_hour_time_binning:

.. _one_hour_time_binning:

one_hour_time_binning Function
===============================

.. function:: one_hour_time_binning(data_frame)

   This function bins the pickup times into 1-hour intervals and aggregates trip counts for each zone.

   :param data_frame: A pandas DataFrame containing taxi data.
   :type data_frame: pandas.DataFrame
   :return: A pandas DataFrame with trip counts aggregated by zone and 1-hour time intervals.
   :rtype: pandas.DataFrame

   This function takes a DataFrame ``data_frame`` containing taxi data and processes it by binning the pickup times into 1-hour intervals. It first converts the 'Trip_Pickup_DateTime' column to datetime format. Then, it defines time bins with 1-hour frequency spanning from the minimum to the maximum pickup time. Next, it creates a new column 'time_bin' based on these time bins using the ``pd.cut`` function. After binning, the function returns the processed DataFrame.

.. _make_forecast:

make_forecast Function
=======================

.. function:: make_forecast(model, df)

   This function makes a forecast using the given model and DataFrame.

   :param model: A Prophet model used for forecasting.
   :type model: Prophet
   :param df: A pandas DataFrame containing historical data.
   :type df: pandas.DataFrame
   :return: A pandas DataFrame with the forecast.
   :rtype: pandas.DataFrame

   This function generates a forecast using the provided Prophet model and historical data DataFrame ``df``. It first creates a future DataFrame with a specified number of periods (28 days * 24 hours per day), excluding the historical data, using the ``make_future_dataframe`` method of the model. Then, it generates the forecast using the ``predict`` method of the model. The resulting forecast DataFrame is returned.

.. _tune_prophet_parameters:

tune_prophet_parameters Function
=================================

.. function:: tune_prophet_parameters(df, param_grid)

   This function tunes Prophet parameters and returns the best parameters.

   :param df: A pandas DataFrame containing historical data.
   :type df: pandas.DataFrame
   :param param_grid: A dictionary containing parameter names as keys and lists of parameter values to be tuned as values.
   :type param_grid: dict
   :return: The best parameters for the Prophet model.
   :rtype: dict

   This function tunes Prophet parameters using cross-validation on the provided historical data DataFrame ``df``. It iterates over all possible combinations of parameters specified in ``param_grid``, fits a Prophet model with each combination, performs cross-validation, calculates the Mean Absolute Percentage Error (MAPE) or Mean Squared Error (MSE) as a performance metric, and returns the parameters resulting in the lowest error.

.. _calculate_mape:

calculate_mape Function
========================

.. function:: calculate_mape(table, forecast_column, actual_column)

   This function calculates the Mean Absolute Percentage Error (MAPE) between forecasted and actual values.

   :param table: A pandas DataFrame containing forecasted and actual values.
   :type table: pandas.DataFrame
   :param forecast_column: The name of the column containing forecasted values.
   :type forecast_column: str
   :param actual_column: The name of the column containing actual values.
   :type actual_column: str
   :return: The calculated MAPE.
   :rtype: float

   This function computes the MAPE between the forecasted values in the column ``forecast_column`` and the actual values in the column ``actual_column`` of the DataFrame ``table``. It returns the calculated MAPE as a float value.



