
EDA
===========

To sum up the EDA will test the factors Stationarity, Trend and Saisonality for each zone. Since the trend is mostly better seen visualy the function test for Stationarity and Seasonality. 

.. function:: analyze_stationarity_seasonality(data)

   Analyze the stationarity and seasonality of the demand data for each zone.

   :param data: A pandas DataFrame containing taxi demand data with a 'time_bin' column and a 'zone' column.
   :type data: pandas.DataFrame
   :return: A DataFrame summarizing the stationarity and seasonality analysis for each zone.
   :rtype: pandas.DataFrame

   This function analyzes the stationarity and seasonality of the demand data for each zone. It first sets the 'time_bin' column as the index of the DataFrame. For each unique zone in the 'zone' column, it performs the following analyses:
   
   - **ADF Test (Augmented Dickey-Fuller)**: Determines if the time series is stationary. A p-value less than 0.05 indicates stationarity.
   - **KPSS Test (Kwiatkowski-Phillips-Schmidt-Shin)**: Tests for stationarity. A p-value greater than 0.05 indicates stationarity.
   - **Seasonal Component Extraction**: Uses Seasonal-Trend decomposition using LOESS (STL) to extract the seasonal component. If the seasonal component is not entirely NaN, the series is considered seasonal.

   The results for each zone are compiled into a DataFrame with the following columns: 'Zone', 'ADF Statistic', 'P-value (ADF)', 'KPSS Statistic', 'P-value (KPSS)', 'Is Stationary', and 'Seasonality'.


      .. code-block:: python

                     import pandas as pd
                     from statsmodels.tsa.stattools import adfuller, kpss
                     from statsmodels.tsa.seasonal import STL
               
                     def analyze_stationarity_seasonality(data):
                         data.set_index('time_bin', inplace=True)
                         results = []
               
                         for zone in data['zone'].unique():
                             subset = data[data['zone'] == zone]['demand']
                             adf_stat, adf_p = adfuller(subset, autolag='AIC')[:2]
                             kpss_stat, kpss_p = kpss(subset, regression='c')[:2]
                             seasonal_component = STL(subset, seasonal=13).fit().seasonal
                             seasonality = 'Yes' if not seasonal_component.isna().all() else 'No'
                             is_stationary = 'Stationary' if adf_p &lt; 0.05 and kpss_p &gt; 0.05 else 'Non-Stationary'
               
                             results.append([zone, adf_stat, adf_p, kpss_stat, kpss_p, is_stationary, seasonality])
               
                         return pd.DataFrame(results, columns=['Zone', 'ADF Statistic', 'P-value (ADF)', 'KPSS Statistic', 'P-value (KPSS)', 'Is Stationary', 'Seasonality'])



