Dashboard
============================
The interactive nature of the dashboard allows users to explore data dynamically, making it easier to pinpoint problem areas, identify opportunities for growth, and make data-driven decisions. Whether it's understanding peak demand times, analyzing geographical hotspots for taxi pickups and drop-offs, or forecasting future trends, the dashboard is designed to support various strategic and operational needs of taxi companies.
By integrating geospatial data, historical trends, and predictive models, the dashboard offers a holistic view of taxi demand and performance. This comprehensive approach not only enhances the understanding of current and past trends but also provides actionable insights for future planning and optimization.

Dashboard Database
-------------------
Our Tableau database is structured to integrate three distinct data sources, each serving a unique purpose for comprehensive analysis and visualization. In Tableau, data sources are generally merged together using joins, blends, or unions, depending on the nature of the data and the desired analysis. For our dashboard we have applied data blending to combine data from different sources, linking them on a common dimension to create a combined view while maintaining separate data connections.

**Forecast Data:** The first data source is the forecast data for February 2009, which provides predictive insights and future trends based on our forecasting model.

**Historical Data:** The second data source comprises historical data for January, encompassing past performance metrics, actual values, and historical trends, offering a baseline for comparative analysis against the forecasted data.

**Geo Data:** The third data source is GeoJSON data specifically for New York City, which includes detailed geographical information such as coordinates, boundaries, and spatial attributes essential for mapping and location-based analysis.

Dashboard Structure
----------------------
Our dashboard consists of four key worksheets designed to provide a comprehensive and interactive analysis of taxi demand in Manhattan: Manhattan Map, Historical Weekday Analysis, February Forecast, and February Weekly Forecast.

**Manhattan Geomap:** The Manhattan Map is a geomap showcasing demand as a heatmap, highlighting the intensity across various districts of Manhattan and displaying the city's roads for context.

**Historical Weekday Analysis:** The Historical Weekday Analysis worksheet presents a heatmap of January's past demand, broken down by weekday for each Manhattan district. 

**February Forecast:** The February Forecast worksheet utilizes a bar chart to depict the projected demand for the entire month of February.

**February Weekly Forecast:** The February Weekly Forecast offers a heatmap illustrating the forecasted weekly demand throughout February.

These sheets are integrated into a single, interactive dashboard, interconnected through an action filter based on the 'zone,' representing the districts of Manhattan. This setup allows users to seamlessly explore and analyze demand patterns across different timeframes and geographical areas.
