Origin Destiny Forecast
=============================

If you now look at the pickups of the past and their dropoff points, you can create a matrix that shows in what proportion which pickup went where. In theory, you could now multiply the matrix of distributions by the new forecast, which would also allow you to predict the final destinations of the forecast pickups. 

This is of course subject to the assumption that the distributions remain constant over time and zones:

.. math::

   D_{ijt} = D_{it} \times OD_{ijt}

where:

- \( D_{ijt} \) represents the Demand for the trip from start zone \( i \) to end zone \( j \) at time \( t \).
- \( D_{it} \) represents the Demandd for all trips starting from zone \( i \) at time \( t \).
- \( OD_{ijt} \) represents the number of trips from start zone \( i \) to end zone \( j \) at time \( t \).



A simple way would be to compare the predicted OD pairs with the real pairs and calculate the loss function. 

As we can see in the Heatmaps of Chapter 3 here the distributions are not constant: https://colab.research.google.com/drive/1VaifCYGzjBQ9H1SxjRtiXDMi6PvxZxRb#scrollTo=-t71LCPxJs3w. 

As the effort involved was greater than the benefit, the approach only went as far as creating the OD pairs. An evaluation would not have been expedient for our Project. 

Therefore, only the creation of the OD pairs is explained below. 

Function: Get OD Pairs
------------------------------------------

.. function:: getODPairs(hourly_trip_counts, forecast)

   Obtain Origin-Destination (OD) pairs from hourly trip counts and forecast data.

   :param hourly_trip_counts: DataFrame containing hourly trip counts with columns 'start_zone', 'end_zone', 'DateTime', and 'Trips'.
   :type hourly_trip_counts: pandas.DataFrame
   :param forecast: DataFrame containing forecast data with columns 'forecast_origin', 'start_zone', 'Hour', 'Day'.
   :type forecast: pandas.DataFrame
   :return: DataFrame containing OD pairs with forecasted trip counts.
   :rtype: pandas.DataFrame

   This function calculates Origin-Destination (OD) pairs from hourly trip counts and forecast data. It extracts the hour of the day and day of the month from the 'DateTime' column of the hourly trip counts DataFrame. Then, it creates a cross-tabulation of trip counts grouped by days, hours, and end zones. Next, it calculates the percentage values for each zone in the cross-tabulation. The forecast DataFrame is merged with the percentage cross-tabulation based on start zone, day, and hour.

   - The function then multiplies the forecasted values by the percentage values of each end zone to obtain forecasted trip counts for each OD pair.
   - Finally, it rounds the resulting data and returns a DataFrame containing the OD pairs with forecasted trip counts.

   This function is useful for generating OD pairs with forecasted trip counts, which can be used for further analysis and planning in transportation systems.

.. code-block:: python

    def getODPairs(hourly_trip_counts, forecast):

        # Extract the hours of the day and the days of the month from the DateTime column
        hourly_trip_counts['Hour'] = pd.to_datetime(hourly_trip_counts['DateTime']).dt.hour
        hourly_trip_counts['Day'] = pd.to_datetime(hourly_trip_counts['DateTime']).dt.day

        # Create a cross-tabulation with the number of trips, grouped by days, hours, and end_zone
        cross_table = pd.crosstab([hourly_trip_counts['start_zone'], hourly_trip_counts['Day'], hourly_trip_counts['Hour']],
                                   hourly_trip_counts['end_zone'],
                                   values=hourly_trip_counts['Trips'],
                                   aggfunc='sum',
                                   margins=True,
                                   margins_name='Total').fillna(0)

        # Calculate the percentage values
        cross_table_percent = cross_table.div(cross_table['Total'], axis=0).fillna(0)

        # Merge the forecast DataFrame with the percentage cross-tabulation
        merged_data = pd.merge(forecast, cross_table_percent,
                               how='inner',
                               left_on=['start_zone', 'Day', 'Hour'],
                               right_index=True).drop(columns=['Total'])

        # Multiply the forecast values by the percentage values
        for end_zone in cross_table_percent.columns[:-1]:  # Exclude 'Total' column
            merged_data[end_zone] = merged_data['forecast_origin'] * merged_data[end_zone]

        # Round the results
        rounded_data = merged_data.round(0)

        return rounded_data
