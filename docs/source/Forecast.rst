.. _process_taxi_data:

After a basic Data cleaning we want to know in which region how many pickups took place.
Our Raw Data only contains the Info which Picktup had which pickup Coordinates. 

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



