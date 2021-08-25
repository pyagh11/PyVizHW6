# San Francisco Rental Prices Dashboard

In this notebook, you will compile the visualizations from the previous analysis into functions that can be used for a Panel dashboard.


```python
# imports
import panel as pn
pn.extension('plotly')
import plotly.express as px
import pandas as pd
import hvplot.pandas
import matplotlib.pyplot as plt
import os
from pathlib import Path
from dotenv import load_dotenv
```


```python
# Read the Mapbox API key
load_dotenv("MY_Keys.env")
map_box_api = os.getenv("MAPBOX_API_KEY")
px.set_mapbox_access_token(map_box_api)
```


```python
map_box_api
```




    'pk.eyJ1IjoicHlhZ2gxMSIsImEiOiJja250bXl3dW4wM3U5Mm9xcWFreWxjY2FnIn0.Y9jjk92mGuiiSK2s68Pjqg'



# Import Data


```python
# Import the necessary CSVs to Pandas DataFrames
# YOUR CODE HERE!
file_path = Path("Data/sfo_neighborhoods_census_data.csv")
sfo_data = pd.read_csv(file_path, index_col="year")

file_path = Path("Data/neighborhoods_coordinates.csv")
df_neighborhood_locations = pd.read_csv(file_path)

```


```python
sfo_data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>neighborhood</th>
      <th>sale_price_sqr_foot</th>
      <th>housing_units</th>
      <th>gross_rent</th>
    </tr>
    <tr>
      <th>year</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2010</th>
      <td>Alamo Square</td>
      <td>291.182945</td>
      <td>372560</td>
      <td>1239</td>
    </tr>
    <tr>
      <th>2010</th>
      <td>Anza Vista</td>
      <td>267.932583</td>
      <td>372560</td>
      <td>1239</td>
    </tr>
    <tr>
      <th>2010</th>
      <td>Bayview</td>
      <td>170.098665</td>
      <td>372560</td>
      <td>1239</td>
    </tr>
    <tr>
      <th>2010</th>
      <td>Buena Vista Park</td>
      <td>347.394919</td>
      <td>372560</td>
      <td>1239</td>
    </tr>
    <tr>
      <th>2010</th>
      <td>Central Richmond</td>
      <td>319.027623</td>
      <td>372560</td>
      <td>1239</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_neighborhood_locations.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Neighborhood</th>
      <th>Lat</th>
      <th>Lon</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Alamo Square</td>
      <td>37.791012</td>
      <td>-122.402100</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Anza Vista</td>
      <td>37.779598</td>
      <td>-122.443451</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Bayview</td>
      <td>37.734670</td>
      <td>-122.401060</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Bayview Heights</td>
      <td>37.728740</td>
      <td>-122.410980</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Bernal Heights</td>
      <td>37.728630</td>
      <td>-122.443050</td>
    </tr>
  </tbody>
</table>
</div>



- - -

## Panel Visualizations

In this section, you will copy the code for each plot type from your analysis notebook and place it into separate functions that Panel can use to create panes for the dashboard. 

These functions will convert the plot object to a Panel pane.

Be sure to include any DataFrame transformation/manipulation code required along with the plotting code.

Return a Panel pane object from each function that can be used to build the dashboard.

Note: Remove any `.show()` lines from the code. We want to return the plots instead of showing them. The Panel dashboard will then display the plots.


```python
# Define Panel Visualization Functions
def housing_units_per_year():
    """Housing Units Per Year."""
    
    housing_unit= sfo_data.groupby('year').mean()
    housing_units = housing_unit.drop(['sale_price_sqr_foot', 'gross_rent'], axis = 1)
    
    #housing_units_fig = plt.figure()
    # housing_units_per_year_plot = plt.figure()
    min = housing_units.min()['housing_units']
    max = housing_units.max()['housing_units']
    
    housing_units_fig = plt.figure()
    housing_units_per_year_plot = housing_units.plot.bar(ylim =(min-2000, max+2000),title="Average Housing Units/Year in San Francisco", figsize=(12,8))
    
    plt.close(housing_units_fig)

    #plt.close(housing_units_per_year_plot)
    
    
    # return pn.pane.Matplotlib(housing_units_per_year_plot)
    return pn.pane.Matplotlib(housing_units_fig)


def average_gross_rent():
    """Average Gross Rent in San Francisco Per Year."""
    
    avg_gross_rent = pd.DataFrame(sfo_data.groupby(['year']).mean()['gross_rent'])
    
    avg_gross_rent_plot = avg_gross_rent.plot.line(title="Average Gross Rent by Year in San Francisco", figsize=(12,8), color='red')

    return avg_gross_rent_plot

def average_sales_price():
    """Average Sales Price Per Year."""
    
    avg_price_sqft = pd.DataFrame(sfo_data.groupby(['year']).mean()['sale_price_sqr_foot'])
    average_sales_price_plot = avg_price_sqft.plot.line(title="Average Price per SqFt by year in San Francisco",figsize=(12,8),color='purple')
    
    return average_sales_price_plot 

def average_price_by_neighborhood():
    """Average Prices by Neighborhood."""
    
    mean_values = sfo_data.groupby([sfo_data.index, "neighborhood"]).mean()
    mean_values.reset_index(inplace=True)
    # mean_values.rename(columns={"level_0": "year"}, inplace=True)
    
    average_price_by_neighborhood_plot = mean_values.hvplot.line(
    x="year",
    y="sale_price_sqr_foot",
    xlabel= "Year",
    ylabel="Average Sale Price per Square Foot",
    groupby="neighborhood",
)

    return average_price_by_neighborhood_plot

def top_most_expensive_neighborhoods():
    """Top 10 Most Expensive Neighborhoods."""

    ten_most_expensive_df = sfo_data.groupby("neighborhood").mean()
    ten_most_expensive_df = ten_most_expensive_df.sort_values("sale_price_sqr_foot", ascending=False).head(10)
    ten_most_expensive_df = ten_most_expensive_df.reset_index()
    
    top_most_expensive_neighborhoods_plot = ten_most_expensive_df.hvplot.bar(
    x="neighborhood",
    y="sale_price_sqr_foot",
    title="Top 10 Most Expensive Neighborhoods in SFO",
    xlabel="Neighborhood",
    ylabel="Average Sale Price per Square Foot",
    height=400,
    rot=45
    )

    return top_most_expensive_neighborhoods_plot

def most_expensive_neighborhoods_rent_sales():
    """Comparison of Rent and Sales Prices of Most Expensive Neighborhoods."""   
    
    most_expensive_neighborhoods_rent_sales_plot = sfo_data.hvplot.bar(
    x="year",
    y=["gross_rent", "sale_price_sqr_foot"],
    title="Top 10 Most Expensive Neighborhoods in SFO",
    xlabel="Neighborhood",
    ylabel="Num Housing Units",
    height=400,
    rot=45,
    groupby="neighborhood"
)

    
    return most_expensive_neighborhoods_rent_sales_plot

# def parallel_coordinates():  commented out due to optional
   # """Parallel Coordinates Plot."""

    # YOUR CODE HERE!



# def parallel_categories(): commented out due to optional
 #   """Parallel Categories Plot."""
    
    # YOUR CODE HERE!



def neighborhood_map():
    """Neighborhood Map."""

    mean_neighborhoods = sfo_data.groupby("neighborhood").mean()
    mean_neighborhoods = mean_neighborhoods.reset_index()
    
    ten_most_expensive_df = sfo_data.groupby("neighborhood").mean()
    ten_most_expensive_df = ten_most_expensive_df.sort_values("sale_price_sqr_foot", ascending=False).head(10)
    ten_most_expensive_df = ten_most_expensive_df.reset_index()

    values_and_locations_df = pd.concat([
    df_neighborhood_locations,
    mean_neighborhoods['sale_price_sqr_foot'],
    mean_neighborhoods['housing_units'],
    mean_neighborhoods['gross_rent']
    ], axis=1).dropna()

    values_and_locations_df.head()
    
    px.set_mapbox_access_token(map_box_api)
    
    map = px.scatter_mapbox(
    values_and_locations_df,
    lat="Lat",
    lon="Lon",
    size="sale_price_sqr_foot",
    color="gross_rent",
    color_continuous_scale=px.colors.cyclical.IceFire,
    size_max=15,
    zoom=11,
    width=1100,
    hover_name="Neighborhood",
    title="Avg Sale price per Square Foot and Gross Rent in San Francisco",
    )
    

    plotly_panel = pn.pane.Plotly(map)
    plotly_panel._updates = True
    return plotly_panel

# def sunburst(): commented out due to optional
  #  """Sunburst Plot."""
    
    # YOUR CODE HERE!

```

## Panel Dashboard

In this section, you will combine all of the plots into a single dashboard view using Panel. Be creative with your dashboard design!


```python
# Create a Title for the Dashboard
# YOUR CODE HERE!
title = pn.pane.Markdown(
    """
# Real Estate Analysis of San Francisco from 2010 to 2016
""",
    width=800,
)
welcome = pn.pane.Markdown(
    """
This dashboard presents a visual analysis of the San Francisco real estate market.
"""
)

# Create a tab layout for the dashboard
# YOUR CODE HERE!

tabs = pn.Tabs(
    ("Welcome", pn.Column(welcome, neighborhood_map())),
    ("Yearly Market Analysis", pn.Row(housing_units_per_year(), average_gross_rent(), average_sales_price())),
    ("Neighborhood Analysis", pn.Column(average_price_by_neighborhood(), top_most_expensive_neighborhoods(), most_expensive_neighborhoods_rent_sales())),
)
panel = pn.Column(pn.Row(title), tabs, width=9000)
# Create the dashboard
# YOUR CODE HERE!
```


    
![png](output_files/output_12_0.png)
    



    
![png](output_files/output_12_1.png)
    



    
![png](output_files/output_12_2.png)
    


## Serve the Panel Dashboard


```python
# Serve the# dashboard
# YOUR CODE HERE!
panel.servable()
```








<div id='8819'>





  <div class="bk-root" id="9a60e7af-b7e6-4de2-abe2-e6fd0a9cbc6e" data-root-id="8819"></div>
</div>
<script type="application/javascript">(function(root) {
  function embed_document(root) {
    var docs_json = {"f8ac257a-60a0-433c-a63e-a56a7ea3b5d6":{"roots":{"references":[{"attributes":{"child":{"id":"8828"},"name":"Row13770","title":"Yearly Market Analysis"},"id":"8832","type":"Panel"},{"attributes":{},"id":"8927","type":"ResetTool"},{"attributes":{"child":{"id":"8833"},"name":"Column13917","title":"Neighborhood Analysis"},"id":"9034","type":"Panel"},{"attributes":{"axis_label":"Neighborhood","bounds":"auto","formatter":{"id":"8945"},"major_label_orientation":0.7853981633974483,"ticker":{"id":"8917"}},"id":"8916","type":"CategoricalAxis"},{"attributes":{},"id":"8914","type":"LinearScale"},{"attributes":{"axis_label":"Num Housing Units","bounds":"auto","formatter":{"id":"9008"},"major_label_orientation":"horizontal","ticker":{"id":"8981"}},"id":"8980","type":"LinearAxis"},{"attributes":{},"id":"8869","type":"Selection"},{"attributes":{"end":929.3801355198136,"reset_end":644.0175329447045,"reset_start":141.1976609302527,"start":0.0,"tags":[[["sale_price_sqr_foot","sale_price_sqr_foot",null]]]},"id":"8836","type":"Range1d"},{"attributes":{"css_classes":["markdown"],"margin":[5,5,5,5],"name":"Markdown13757","text":"&lt;h1&gt;Real Estate Analysis of San Francisco from 2010 to 2016&lt;/h1&gt;","width":800},"id":"8821","type":"panel.models.markup.HTML"},{"attributes":{},"id":"8946","type":"BasicTickFormatter"},{"attributes":{},"id":"8984","type":"SaveTool"},{"attributes":{"line_color":"#30a2da","line_width":2,"x":{"field":"year"},"y":{"field":"sale_price_sqr_foot"}},"id":"8871","type":"Line"},{"attributes":{"below":[{"id":"8977"}],"center":[{"id":"8979"},{"id":"8983"}],"left":[{"id":"8980"}],"margin":[5,5,5,5],"min_border_bottom":10,"min_border_left":10,"min_border_right":10,"min_border_top":10,"plot_height":400,"plot_width":700,"renderers":[{"id":"9004"}],"sizing_mode":"fixed","title":{"id":"8969"},"toolbar":{"id":"8990"},"x_range":{"id":"8965"},"x_scale":{"id":"8973"},"y_range":{"id":"8966"},"y_scale":{"id":"8975"}},"id":"8968","subtype":"Figure","type":"Plot"},{"attributes":{"children":[{"id":"8838"},{"id":"8900"}],"margin":[0,0,0,0],"name":"Row13895"},"id":"8834","type":"Row"},{"attributes":{},"id":"8925","type":"WheelZoomTool"},{"attributes":{"callback":null,"renderers":[{"id":"9004"}],"tags":["hv_created"],"tooltips":[["year","@{year}"],["Variable","@{Variable}"],["value","@{value}"]]},"id":"8967","type":"HoverTool"},{"attributes":{"axis":{"id":"8916"},"grid_line_color":null,"ticker":null},"id":"8918","type":"Grid"},{"attributes":{},"id":"8878","type":"BasicTickFormatter"},{"attributes":{"factors":["Union Square District","Merced Heights","Miraloma Park","Pacific Heights","Westwood Park","Telegraph Hill","Presidio Heights","Cow Hollow","Potrero Hill","South Beach"],"tags":[[["neighborhood","neighborhood",null]]]},"id":"8904","type":"FactorRange"},{"attributes":{"active_drag":"auto","active_inspect":"auto","active_multi":null,"active_scroll":"auto","active_tap":"auto","tools":[{"id":"8837"},{"id":"8855"},{"id":"8856"},{"id":"8857"},{"id":"8858"},{"id":"8859"}]},"id":"8861","type":"Toolbar"},{"attributes":{"fill_color":{"field":"Variable","transform":{"id":"8997"}},"top":{"field":"value"},"width":{"value":0.8},"x":{"field":"xoffsets"}},"id":"9001","type":"VBar"},{"attributes":{"fill_alpha":{"value":0.2},"fill_color":{"field":"Variable","transform":{"id":"8997"}},"line_alpha":{"value":0.2},"top":{"field":"value"},"width":{"value":0.8},"x":{"field":"xoffsets"}},"id":"9003","type":"VBar"},{"attributes":{"text":"Top 10 Most Expensive Neighborhoods in SFO","text_color":{"value":"black"},"text_font_size":{"value":"12pt"}},"id":"8908","type":"Title"},{"attributes":{},"id":"9037","type":"Selection"},{"attributes":{"fill_color":{"value":"#30a2da"},"top":{"field":"sale_price_sqr_foot"},"width":{"value":0.8},"x":{"field":"neighborhood"}},"id":"8939","type":"VBar"},{"attributes":{},"id":"9008","type":"BasicTickFormatter"},{"attributes":{},"id":"8975","type":"LinearScale"},{"attributes":{"source":{"id":"8998"}},"id":"9005","type":"CDSView"},{"attributes":{},"id":"8890","type":"UnionRenderers"},{"attributes":{"data_source":{"id":"8936"},"glyph":{"id":"8939"},"hover_glyph":null,"muted_glyph":{"id":"8941"},"nonselection_glyph":{"id":"8940"},"selection_glyph":null,"view":{"id":"8943"}},"id":"8942","type":"GlyphRenderer"},{"attributes":{"children":[{"id":"9031"},{"id":"9033"}],"margin":[0,0,0,0],"name":"Column13916"},"id":"9030","type":"Column"},{"attributes":{},"id":"9007","type":"CategoricalTickFormatter"},{"attributes":{"axis_label":"Neighborhood","bounds":"auto","formatter":{"id":"9007"},"major_label_orientation":0.7853981633974483,"ticker":{"id":"8978"}},"id":"8977","type":"CategoricalAxis"},{"attributes":{"source":{"id":"8868"}},"id":"8875","type":"CDSView"},{"attributes":{"line_alpha":0.1,"line_color":"#30a2da","line_width":2,"x":{"field":"year"},"y":{"field":"sale_price_sqr_foot"}},"id":"8872","type":"Line"},{"attributes":{},"id":"8852","type":"BasicTicker"},{"attributes":{},"id":"8912","type":"CategoricalScale"},{"attributes":{},"id":"8855","type":"SaveTool"},{"attributes":{"bottom_units":"screen","fill_alpha":0.5,"fill_color":"lightgrey","left_units":"screen","level":"overlay","line_alpha":1.0,"line_color":"black","line_dash":[4,4],"line_width":2,"right_units":"screen","top_units":"screen"},"id":"8989","type":"BoxAnnotation"},{"attributes":{"_render_count":0,"config":null,"data":[{"hovertemplate":"<b>%{hovertext}</b><br><br>sale_price_sqr_foot=%{marker.size}<br>Lat=%{lat}<br>Lon=%{lon}<br>gross_rent=%{marker.color}<extra></extra>","legendgroup":"","marker":{"coloraxis":"coloraxis","sizemode":"area","sizeref":4.017747811875842},"mode":"markers","name":"","showlegend":false,"subplot":"mapbox","type":"scattermapbox"}],"data_sources":[{"id":"8825"}],"layout":{"coloraxis":{"colorbar":{"title":{"text":"gross_rent"}},"colorscale":[[0.0,"#000000"],[0.0625,"#001f4d"],[0.125,"#003786"],[0.1875,"#0e58a8"],[0.25,"#217eb8"],[0.3125,"#30a4ca"],[0.375,"#54c8df"],[0.4375,"#9be4ef"],[0.5,"#e1e9d1"],[0.5625,"#f3d573"],[0.625,"#e7b000"],[0.6875,"#da8200"],[0.75,"#c65400"],[0.8125,"#ac2301"],[0.875,"#820000"],[0.9375,"#4c0000"],[1.0,"#000000"]]},"legend":{"itemsizing":"constant","tracegroupgap":0},"mapbox":{"accesstoken":"pk.eyJ1IjoicHlhZ2gxMSIsImEiOiJja250bXl3dW4wM3U5Mm9xcWFreWxjY2FnIn0.Y9jjk92mGuiiSK2s68Pjqg","center":{"lat":37.76019350684932,"lon":-122.43912380821916},"domain":{"x":[0.0,1.0],"y":[0.0,1.0]},"zoom":11},"template":{"data":{"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5}},"type":"bar"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5}},"type":"barpolar"}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"choropleth":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"choropleth"}],"contour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"contour"}],"contourcarpet":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"contourcarpet"}],"heatmap":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmap"}],"heatmapgl":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmapgl"}],"histogram":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"histogram"}],"histogram2d":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2d"}],"histogram2dcontour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2dcontour"}],"mesh3d":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"mesh3d"}],"parcoords":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"parcoords"}],"pie":[{"automargin":true,"type":"pie"}],"scatter":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter"}],"scatter3d":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter3d"}],"scattercarpet":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattercarpet"}],"scattergeo":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergeo"}],"scattergl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergl"}],"scattermapbox":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattermapbox"}],"scatterpolar":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolar"}],"scatterpolargl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolargl"}],"scatterternary":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterternary"}],"surface":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"surface"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}]},"layout":{"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"autotypenumbers":"strict","coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]],"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]},"colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"geo":{"bgcolor":"white","lakecolor":"white","landcolor":"#E5ECF6","showlakes":true,"showland":true,"subunitcolor":"white"},"hoverlabel":{"align":"left"},"hovermode":"closest","mapbox":{"style":"light"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"ternary":{"aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"title":{"x":0.05},"xaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2},"yaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2}}},"title":{"text":"Avg Sale price per Square Foot and Gross Rent in San Francisco"},"width":1100},"margin":[5,5,5,5],"name":"Plotly13761","viewport_update_throttle":200},"id":"8826","type":"panel.models.plotly.PlotlyPlot"},{"attributes":{"data":{"sale_price_sqr_foot":{"__ndarray__":"Dkc7WO0yckCafszcbwhxQIGRs5ot42ZAJzog0LQ8eEAWMinKGEd+QI0V5FDt0IJAVCHcmLVPdUA=","dtype":"float64","order":"little","shape":[7]},"year":[2010,2011,2012,2013,2014,2015,2016]},"selected":{"id":"8869"},"selection_policy":{"id":"8890"}},"id":"8868","type":"ColumnDataSource"},{"attributes":{"factors":["gross_rent","sale_price_sqr_foot"],"palette":["#30a2da","#fc4f30"]},"id":"8997","type":"CategoricalColorMapper"},{"attributes":{"child":{"id":"8823"},"name":"Column13763","title":"Welcome"},"id":"8827","type":"Panel"},{"attributes":{"source":{"id":"8936"}},"id":"8943","type":"CDSView"},{"attributes":{"axis":{"id":"8847"},"grid_line_color":null,"ticker":null},"id":"8850","type":"Grid"},{"attributes":{"fill_alpha":{"value":0.1},"fill_color":{"value":"#30a2da"},"line_alpha":{"value":0.1},"top":{"field":"sale_price_sqr_foot"},"width":{"value":0.8},"x":{"field":"neighborhood"}},"id":"8940","type":"VBar"},{"attributes":{"overlay":{"id":"8989"}},"id":"8987","type":"BoxZoomTool"},{"attributes":{"children":[{"id":"8821"}],"margin":[0,0,0,0],"name":"Row13919"},"id":"8820","type":"Row"},{"attributes":{},"id":"8848","type":"BasicTicker"},{"attributes":{"data":{"hovertext":[["Alamo Square","Anza Vista","Bayview","Bayview Heights","Bernal Heights","Buena Vista Park","Central Richmond","Central Sunset","Clarendon Heights","Corona Heights","Cow Hollow","Croker Amazon","Diamond Heights","Downtown","Duboce Triangle","Eureka Valley/Dolores Heights","Excelsior","Financial District North","Financial District South","Forest Knolls","Glen Park","Golden Gate Heights","Haight Ashbury","Hayes Valley","Hunters Point","Ingleside","Ingleside Heights","Inner Mission","Inner Parkside","Inner Richmond","Inner Sunset","Jordan Park/Laurel Heights","Lake --The Presidio","Lone Mountain","Lower Pacific Heights","Marina","Merced Heights","Midtown Terrace","Miraloma Park","Mission Bay","Mission Dolores","Mission Terrace","Nob Hill","Noe Valley","North Beach","North Waterfront","Oceanview","Outer Mission","Outer Parkside","Outer Richmond","Outer Sunset","Pacific Heights","Park North","Parkside","Parnassus/Ashbury Heights","Portola","Potrero Hill","Presidio Heights","Russian Hill","Silver Terrace","South Beach","South of Market","Sunnyside","Telegraph Hill","Twin Peaks","Union Square District","Van Ness/ Civic Center","Visitacion Valley","West Portal","Western Addition","Westwood Highlands","Westwood Park","Yerba Buena"]],"lat":[{"__ndarray__":"LV+X4T/lQkB1AwXeyeNCQBmto6oJ3kJAvqQxWkfdQkA2cXK/Q91CQDQMHxFT4kJAVvFG5pHjQkCVYHE4899CQPjfSnZs4EJAcQM+P4zkQkDoMF9egOVCQD/G3LWE3EJANnFyv0PdQkCMZ9DQP+VCQMPYQpCD4kJA8WjjiLXgQkC+pDFaR91CQIxn0NA/5UJAjGfQ0D/lQkAMjpJX5+BCQDZxcr9D3UJAe5+qQgPhQkDLoUW28+FCQMuhRbbz4UJA4nX9gt3cQkBIE+8AT9xCQHiXi/hO3EJAxvmbUIjgQkCVYHE4899CQAd7E0Ny4kJAlWBxOPPfQkBxAz4/jORCQIOLFTWY5kJAXW3F/rLjQkDoMF9egOVCQIP6ljld5kJAY7SOqibcQkD430p2bOBCQOAtkKD43UJAf59x4UDkQkCO6QlLPOBCQDZxcr9D3UJA6DBfXoDlQkD430p2bOBCQApoImx45kJACmgibHjmQkBjtI6qJtxCQD/G3LWE3EJAnDOitDfgQkBmg0wycuJCQJVgcTjz30JA6DBfXoDlQkC+h0uOO91CQJVgcTjz30JAL26jAbzhQkAZraOqCd5CQHb9gt2w3UJAcQM+P4zkQkDoMF9egOVCQBmto6oJ3kJAf59x4UDkQkCMZ9DQP+VCQOAtkKD43UJACmgibHjmQkDIW65+bOBCQIxn0NA/5UJA1XPS+8bjQkC+pDFaR91CQLtE9dbA3kJA6DBfXoDlQkCDL0ymCt5CQOAtkKD43UJA6DBfXoDlQkA=","dtype":"float64","order":"little","shape":[73]}],"lon":[{"__ndarray__":"L26jAbyZXsA5mE2AYZxewPfuj/eqmV7A2dMOf02aXsCKH2PuWpxewLWJk/sdnF7AQspPqn2cXsB9VwT/W59ewC2VtyOcnF7Ad76fGi+dXsAge73745tewEPFOH8TnF7Aih9j7lqcXsAvbqMBvJlewOmf4GJFm17AAyFZwAScXsDZ0w5/TZpewC9uowG8mV7AL26jAbyZXsAkr84xIJ1ewIofY+5anF7AJyzxgLKdXsBPl8XE5ptewE+XxcTmm17ApFNXPsuXXsBmu0IfLJ9ewDY3picsn17AzOmymNiaXsB9VwT/W59ewEjdzr7ynl7AWoEhq1ufXsB3vp8aL51ewIjvxKwXnV7APz+MEB6dXsAge73745tewBU1mIbhnV7Aj+TyH9KdXsAtlbcjnJxewGmR7Xw/nV7AHcnlP6SZXsC6ZvLNNptewIofY+5anF7AIHu9++ObXsAtlbcjnJxewKuy74rgmV7Aq7LviuCZXsCP5PIf0p1ewEPFOH8TnF7ABoGVQ4ucXsDp1JXP8p5ewH1XBP9bn17AIHu9++ObXsDHM2jon55ewH1XBP9bn17A7MA5I0qdXsD37o/3qplewDehEAGHmF7Ad76fGi+dXsAge73745tewPfuj/eqmV7AHcnlP6SZXsAvbqMBvJlewGmR7Xw/nV7Aq7LviuCZXsBF14UfnJxewC9uowG8mV7AAkht4uSaXsDZ0w5/TZpewC9RvTWwnV7AIHu9++ObXsD/Qo8YPZ1ewGmR7Xw/nV7AVFxV9l2ZXsA=","dtype":"float64","order":"little","shape":[73]}],"marker.color":[{"__ndarray__":"SZIkSZICpkCrqqqqqq+nQM3MzMzMHKJAAAAAAAA2rUCrqqqqqhCoQKuqqqqqFaVASZIkSZICpkBJkiRJkgKmQAAAAAAAlaFAAAAAAABQo0BJkiRJkgKmQKuqqqqqFaVAAAAAAACAn0BJkiRJkgKmQAAAAACAuKVASZIkSZICpkCrqqqqqq+nQEmSJEmSAqZAAAAAAABQo0AAAAAAANabQAAAAAAAp6ZAzczMzMxSpEBJkiRJkgKmQEmSJEmSAqZAAAAAAAByo0AAAAAAAJqjQAAAAAAAIKdASZIkSZICpkAAAAAAADCpQEmSJEmSAqZASZIkSZICpkBJkiRJkgKmQFVVVVVV9qNAVVVVVVX2o0BJkiRJkgKmQEmSJEmSAqZAAAAAAACsqkAAAAAAAK+kQAAAAACA1qBAVVVVVVXPpEBVVVVVVfajQM3MzMzMyqhASZIkSZICpkBJkiRJkgKmQJqZmZmZXadAzczMzMwEpkAAAAAAAASjQAAAAACAZ6dASZIkSZICpkBJkiRJkgKmQEmSJEmSAqZASZIkSZICpkBJkiRJkgKmQFVVVVVV9qNASZIkSZICpkDNzMzMzByiQEmSJEmSAqZASZIkSZICpkBJkiRJkgKmQAAAAAAAkKtAAAAAAABmoEBJkiRJkgKmQAAAAAAAp6ZASZIkSZICpkBJkiRJkgKmQFVVVVVV9qNASZIkSZICpkAAAAAAAJKsQAAAAAAAp6NAVVVVVVX2o0AAAAAAAJWhQAAAAAAA7q5AVVVVVVX2o0A=","dtype":"float64","order":"little","shape":[73]}],"marker.size":[{"__ndarray__":"cJyd1VTgdkAk1Xd7HVZ3QMu7p//VkmlA/m3ou1d2gkCwicLO+AWCQOHeULPjSnxAkyM5JcKmeEDxRHrAAXt6QGAgnA3rc35AdiCNAlBcgkANV25bts+EQC2aqyMR8HJADjYuKt4ve0BAExo283Z4QAT0IiSWaX9Azb8HR/0RhECMIKw8QUx4QEysfO/MdXhARl0VIGF9fEDnKA0RdB10QMn/9wKcfoNAa+R3ASb7g0BCjahYtxh8QECb/9zsPnZABT/1V/9TZUCYt5qCUv52QPmqSlDEDHhAfmgaicfaeECrA3m3FTuAQN85W0+CqXdAyXixJK7aeUCo3p7PEouAQBsbzgZFn3lA1YI2J6jjfUADLzpc59mAQOt1/4D+PYJAfSK5L8KmiEDjM3wxFTSBQFx/3pp8XohAcr9Bp4mxgUC9Hcljpi56QAabwse6W4BAhQqU+EOjfEAzwd8Vi/OAQA1GhppZunlANYyg6lAkf0A0VzbjjqF0QGONxdbeS25ATpTGpG5QfkAuXEiRaZ59QGJw/yeBo3hAFpsVUHKMhUAyVprHuVt3QBQ4ODjDAnVAYJzjcuzFg0CTVGKj0nF0QJrMCuEbsIRAy835O80ahUBvvf+g3QeDQNMptI5cSWVAya3y7v5QhECG4cLhK9KBQP9hf/GLgoBAvQqFeA0khUCLIC7GYFZ9QCUQFDHyP4xAR1RQI2dCeUCgCCJ5dddyQMr0d9XQJ39AXWt0xv44c0BA3+aooa2AQGzQRFqzeIVAg478xK0FgkA=","dtype":"float64","order":"little","shape":[73]}]},"selected":{"id":"9037"},"selection_policy":{"id":"9036"}},"id":"8825","type":"ColumnDataSource"},{"attributes":{"margin":[5,5,5,5],"name":"Str13768","text":"&lt;pre&gt;AxesSubplot(0.125,0.125;0.775x0.755)&lt;/pre&gt;"},"id":"8831","type":"panel.models.markup.HTML"},{"attributes":{"below":[{"id":"8847"}],"center":[{"id":"8850"},{"id":"8854"}],"left":[{"id":"8851"}],"margin":[5,5,5,5],"min_border_bottom":10,"min_border_left":10,"min_border_right":10,"min_border_top":10,"plot_height":300,"plot_width":700,"renderers":[{"id":"8874"}],"sizing_mode":"fixed","title":{"id":"8839"},"toolbar":{"id":"8861"},"x_range":{"id":"8835"},"x_scale":{"id":"8843"},"y_range":{"id":"8836"},"y_scale":{"id":"8845"}},"id":"8838","subtype":"Figure","type":"Plot"},{"attributes":{},"id":"8985","type":"PanTool"},{"attributes":{},"id":"8999","type":"Selection"},{"attributes":{"active_drag":"auto","active_inspect":"auto","active_multi":null,"active_scroll":"auto","active_tap":"auto","tools":[{"id":"8906"},{"id":"8923"},{"id":"8924"},{"id":"8925"},{"id":"8926"},{"id":"8927"}]},"id":"8929","type":"Toolbar"},{"attributes":{"children":[{"id":"8824"},{"id":"8826"}],"margin":[0,0,0,0],"name":"Column13763"},"id":"8823","type":"Column"},{"attributes":{"children":[{"id":"9032"}],"css_classes":["panel-widget-box"],"margin":[5,5,5,5],"name":"WidgetBox13910"},"id":"9031","type":"Column"},{"attributes":{"margin":[0,0,0,0],"tabs":[{"id":"8827"},{"id":"8832"},{"id":"9034"}]},"id":"8822","type":"Tabs"},{"attributes":{"children":[{"id":"8902"}],"css_classes":["panel-widget-box"],"margin":[5,5,5,5],"name":"WidgetBox13896"},"id":"8901","type":"Column"},{"attributes":{"margin":[20,20,20,20],"min_width":250,"options":["Alamo Square","Anza Vista","Bayview","Buena Vista Park","Central Richmond","Central Sunset","Corona Heights","Cow Hollow","Croker Amazon","Diamond Heights","Downtown ","Eureka Valley/Dolores Heights","Excelsior","Financial District North","Financial District South","Forest Knolls","Glen Park","Golden Gate Heights","Haight Ashbury","Hayes Valley","Hunters Point","Ingleside ","Inner Mission","Inner Parkside","Inner Richmond","Inner Sunset","Jordan Park/Laurel Heights","Lake --The Presidio","Lone Mountain","Lower Pacific Heights","Marina","Miraloma Park","Mission Bay","Mission Dolores","Mission Terrace","Nob Hill","Noe Valley","Oceanview","Outer Parkside","Outer Richmond ","Outer Sunset","Pacific Heights","Park North","Parkside","Parnassus/Ashbury Heights","Portola","Potrero Hill","Presidio Heights","Russian Hill","South Beach","South of Market","Sunnyside","Telegraph Hill","Twin Peaks","Union Square District","Van Ness/ Civic Center","West Portal","Western Addition","Yerba Buena","Bernal Heights ","Clarendon Heights","Duboce Triangle","Ingleside Heights","North Beach","North Waterfront","Outer Mission","Westwood Highlands","Merced Heights","Midtown Terrace","Visitacion Valley","Silver Terrace","Westwood Park","Bayview Heights"],"title":"neighborhood","value":"Alamo Square","width":250},"id":"8902","type":"Select"},{"attributes":{"fill_alpha":{"value":0.1},"fill_color":{"field":"Variable","transform":{"id":"8997"}},"line_alpha":{"value":0.1},"top":{"field":"value"},"width":{"value":0.8},"x":{"field":"xoffsets"}},"id":"9002","type":"VBar"},{"attributes":{"margin":[5,5,5,5],"name":"Str13766","text":"&lt;pre&gt;AxesSubplot(0.125,0.125;0.775x0.755)&lt;/pre&gt;"},"id":"8830","type":"panel.models.markup.HTML"},{"attributes":{},"id":"8917","type":"CategoricalTicker"},{"attributes":{},"id":"8978","type":"CategoricalTicker"},{"attributes":{"data":{"neighborhood":["Union Square District","Merced Heights","Miraloma Park","Pacific Heights","Westwood Park","Telegraph Hill","Presidio Heights","Cow Hollow","Potrero Hill","South Beach"],"sale_price_sqr_foot":{"__ndarray__":"JRAUMfI/jEB9IrkvwqaIQFx/3pp8XohAFpsVUHKMhUBs0ERas3iFQL0KhXgNJIVAy835O80ahUANV25bts+EQJrMCuEbsIRAya3y7v5QhEA=","dtype":"float64","order":"little","shape":[10]}},"selected":{"id":"8937"},"selection_policy":{"id":"8954"}},"id":"8936","type":"ColumnDataSource"},{"attributes":{"overlay":{"id":"8860"}},"id":"8858","type":"BoxZoomTool"},{"attributes":{"children":[{"id":"8834"},{"id":"8907"},{"id":"8964"}],"margin":[0,0,0,0],"name":"Column13917"},"id":"8833","type":"Column"},{"attributes":{"bottom_units":"screen","fill_alpha":0.5,"fill_color":"lightgrey","left_units":"screen","level":"overlay","line_alpha":1.0,"line_color":"black","line_dash":[4,4],"line_width":2,"right_units":"screen","top_units":"screen"},"id":"8860","type":"BoxAnnotation"},{"attributes":{"overlay":{"id":"8928"}},"id":"8926","type":"BoxZoomTool"},{"attributes":{},"id":"8857","type":"WheelZoomTool"},{"attributes":{"margin":[20,20,20,20],"min_width":250,"options":["Alamo Square","Anza Vista","Bayview","Buena Vista Park","Central Richmond","Central Sunset","Corona Heights","Cow Hollow","Croker Amazon","Diamond Heights","Downtown ","Eureka Valley/Dolores Heights","Excelsior","Financial District North","Financial District South","Forest Knolls","Glen Park","Golden Gate Heights","Haight Ashbury","Hayes Valley","Hunters Point","Ingleside ","Inner Mission","Inner Parkside","Inner Richmond","Inner Sunset","Jordan Park/Laurel Heights","Lake --The Presidio","Lone Mountain","Lower Pacific Heights","Marina","Miraloma Park","Mission Bay","Mission Dolores","Mission Terrace","Nob Hill","Noe Valley","Oceanview","Outer Parkside","Outer Richmond ","Outer Sunset","Pacific Heights","Park North","Parkside","Parnassus/Ashbury Heights","Portola","Potrero Hill","Presidio Heights","Russian Hill","South Beach","South of Market","Sunnyside","Telegraph Hill","Twin Peaks","Union Square District","Van Ness/ Civic Center","West Portal","Western Addition","Yerba Buena","Bernal Heights ","Clarendon Heights","Duboce Triangle","Ingleside Heights","North Beach","North Waterfront","Outer Mission","Westwood Highlands","Merced Heights","Midtown Terrace","Visitacion Valley","Silver Terrace","Westwood Park","Bayview Heights"],"title":"neighborhood","value":"Alamo Square","width":250},"id":"9032","type":"Select"},{"attributes":{"data_source":{"id":"8998"},"glyph":{"id":"9001"},"hover_glyph":null,"muted_glyph":{"id":"9003"},"nonselection_glyph":{"id":"9002"},"selection_glyph":null,"view":{"id":"9005"}},"id":"9004","type":"GlyphRenderer"},{"attributes":{"client_comm_id":"f215f1e6ec7a481faad514b1417bfd84","comm_id":"0220f0b20a074e9aad2762b2dd6d3c04","plot_id":"8819"},"id":"9113","type":"panel.models.comm_manager.CommManager"},{"attributes":{"children":[{"id":"8829"},{"id":"8830"},{"id":"8831"}],"margin":[0,0,0,0],"name":"Row13770"},"id":"8828","type":"Row"},{"attributes":{},"id":"8954","type":"UnionRenderers"},{"attributes":{"axis":{"id":"8919"},"dimension":1,"grid_line_color":null,"ticker":null},"id":"8922","type":"Grid"},{"attributes":{},"id":"9036","type":"UnionRenderers"},{"attributes":{},"id":"8923","type":"SaveTool"},{"attributes":{},"id":"8876","type":"BasicTickFormatter"},{"attributes":{},"id":"8937","type":"Selection"},{"attributes":{"axis":{"id":"8977"},"grid_line_color":null,"ticker":null},"id":"8979","type":"Grid"},{"attributes":{},"id":"8973","type":"CategoricalScale"},{"attributes":{"line_alpha":0.2,"line_color":"#30a2da","line_width":2,"x":{"field":"year"},"y":{"field":"sale_price_sqr_foot"}},"id":"8873","type":"Line"},{"attributes":{},"id":"8945","type":"CategoricalTickFormatter"},{"attributes":{},"id":"8859","type":"ResetTool"},{"attributes":{"css_classes":["markdown"],"margin":[5,5,5,5],"name":"Markdown13759","text":"&lt;p&gt;This dashboard presents a visual analysis of the San Francisco real estate market.&lt;/p&gt;"},"id":"8824","type":"panel.models.markup.HTML"},{"attributes":{"axis":{"id":"8851"},"dimension":1,"grid_line_color":null,"ticker":null},"id":"8854","type":"Grid"},{"attributes":{"children":[{"id":"8901"},{"id":"8903"}],"margin":[0,0,0,0],"name":"Column13902"},"id":"8900","type":"Column"},{"attributes":{"active_drag":"auto","active_inspect":"auto","active_multi":null,"active_scroll":"auto","active_tap":"auto","tools":[{"id":"8967"},{"id":"8984"},{"id":"8985"},{"id":"8986"},{"id":"8987"},{"id":"8988"}]},"id":"8990","type":"Toolbar"},{"attributes":{"axis":{"id":"8980"},"dimension":1,"grid_line_color":null,"ticker":null},"id":"8983","type":"Grid"},{"attributes":{"end":2016.0,"reset_end":2016.0,"reset_start":2010.0,"start":2010.0,"tags":[[["year","year",null]]]},"id":"8835","type":"Range1d"},{"attributes":{},"id":"8920","type":"BasicTicker"},{"attributes":{},"id":"8986","type":"WheelZoomTool"},{"attributes":{"text":"Top 10 Most Expensive Neighborhoods in SFO","text_color":{"value":"black"},"text_font_size":{"value":"12pt"}},"id":"8969","type":"Title"},{"attributes":{"children":[{"id":"8968"},{"id":"9030"}],"margin":[0,0,0,0],"name":"Row13909"},"id":"8964","type":"Row"},{"attributes":{"data_source":{"id":"8868"},"glyph":{"id":"8871"},"hover_glyph":null,"muted_glyph":{"id":"8873"},"nonselection_glyph":{"id":"8872"},"selection_glyph":null,"view":{"id":"8875"}},"id":"8874","type":"GlyphRenderer"},{"attributes":{"factors":[["2010","gross_rent"],["2010","sale_price_sqr_foot"],["2011","gross_rent"],["2011","sale_price_sqr_foot"],["2012","gross_rent"],["2012","sale_price_sqr_foot"],["2013","gross_rent"],["2013","sale_price_sqr_foot"],["2014","gross_rent"],["2014","sale_price_sqr_foot"],["2015","gross_rent"],["2015","sale_price_sqr_foot"],["2016","gross_rent"],["2016","sale_price_sqr_foot"]],"tags":[[["year","year",null],["Variable","Variable",null]]]},"id":"8965","type":"FactorRange"},{"attributes":{"below":[{"id":"8916"}],"center":[{"id":"8918"},{"id":"8922"}],"left":[{"id":"8919"}],"margin":[5,5,5,5],"min_border_bottom":10,"min_border_left":10,"min_border_right":10,"min_border_top":10,"plot_height":400,"plot_width":700,"renderers":[{"id":"8942"}],"sizing_mode":"fixed","title":{"id":"8908"},"toolbar":{"id":"8929"},"x_range":{"id":"8904"},"x_scale":{"id":"8912"},"y_range":{"id":"8836"},"y_scale":{"id":"8914"}},"id":"8907","subtype":"Figure","type":"Plot"},{"attributes":{"height":288,"margin":[5,5,5,5],"name":"Matplotlib13764","text":"&lt;img src=&quot;data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAA2AAAAJACAYAAADrSQUmAAAABHNCSVQICAgIfAhkiAAAAAlwSFlzAAAWJQAAFiUBSVIk8AAAADl0RVh0U29mdHdhcmUAbWF0cGxvdGxpYiB2ZXJzaW9uIDMuMC4zLCBodHRwOi8vbWF0cGxvdGxpYi5vcmcvnQurowAADH9JREFUeJzt1zEBwCAQwMBS/54fFYSBOwVZs2ZmPgAAAI77bwcAAAC8woABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQGQD+wUIfMggoFoAAAAASUVORK5CYII=&quot; width=&quot;432px&quot; height=&quot;288px&quot; alt=&quot;&quot;&gt;&lt;/img&gt;","width":432},"id":"8829","type":"panel.models.markup.HTML"},{"attributes":{"fill_alpha":{"value":0.2},"fill_color":{"value":"#30a2da"},"line_alpha":{"value":0.2},"top":{"field":"sale_price_sqr_foot"},"width":{"value":0.8},"x":{"field":"neighborhood"}},"id":"8941","type":"VBar"},{"attributes":{"axis_label":"Average Sale Price per Square Foot","bounds":"auto","formatter":{"id":"8878"},"major_label_orientation":"horizontal","ticker":{"id":"8852"}},"id":"8851","type":"LinearAxis"},{"attributes":{"end":4810.690068306854,"reset_end":4810.690068306854,"reset_start":0.0,"tags":[[["value","value",null]]]},"id":"8966","type":"Range1d"},{"attributes":{},"id":"8924","type":"PanTool"},{"attributes":{"callback":null,"renderers":[{"id":"8874"}],"tags":["hv_created"],"tooltips":[["year","@{year}"],["sale_price_sqr_foot","@{sale_price_sqr_foot}"]]},"id":"8837","type":"HoverTool"},{"attributes":{},"id":"8988","type":"ResetTool"},{"attributes":{},"id":"8981","type":"BasicTicker"},{"attributes":{},"id":"8856","type":"PanTool"},{"attributes":{"axis_label":"Average Sale Price per Square Foot","bounds":"auto","formatter":{"id":"8946"},"major_label_orientation":"horizontal","ticker":{"id":"8920"}},"id":"8919","type":"LinearAxis"},{"attributes":{"children":[{"id":"8820"},{"id":"8822"}],"margin":[0,0,0,0],"min_width":9000,"name":"Column13920","width":9000},"id":"8819","type":"Column"},{"attributes":{"margin":[5,5,5,5],"name":"VSpacer13901","sizing_mode":"stretch_height"},"id":"8903","type":"Spacer"},{"attributes":{"bottom_units":"screen","fill_alpha":0.5,"fill_color":"lightgrey","left_units":"screen","level":"overlay","line_alpha":1.0,"line_color":"black","line_dash":[4,4],"line_width":2,"right_units":"screen","top_units":"screen"},"id":"8928","type":"BoxAnnotation"},{"attributes":{},"id":"8843","type":"LinearScale"},{"attributes":{},"id":"9020","type":"UnionRenderers"},{"attributes":{"axis_label":"Year","bounds":"auto","formatter":{"id":"8876"},"major_label_orientation":"horizontal","ticker":{"id":"8848"}},"id":"8847","type":"LinearAxis"},{"attributes":{"margin":[5,5,5,5],"name":"VSpacer13915","sizing_mode":"stretch_height"},"id":"9033","type":"Spacer"},{"attributes":{"text":"neighborhood: Alamo Square","text_color":{"value":"black"},"text_font_size":{"value":"12pt"}},"id":"8839","type":"Title"},{"attributes":{},"id":"8845","type":"LinearScale"},{"attributes":{"callback":null,"renderers":[{"id":"8942"}],"tags":["hv_created"],"tooltips":[["neighborhood","@{neighborhood}"],["sale_price_sqr_foot","@{sale_price_sqr_foot}"]]},"id":"8906","type":"HoverTool"},{"attributes":{"data":{"Variable":["gross_rent","gross_rent","gross_rent","gross_rent","gross_rent","gross_rent","gross_rent","sale_price_sqr_foot","sale_price_sqr_foot","sale_price_sqr_foot","sale_price_sqr_foot","sale_price_sqr_foot","sale_price_sqr_foot","sale_price_sqr_foot"],"value":{"__ndarray__":"AAAAAABck0AAAAAAAOiXQAAAAAAAKKJAAAAAAAA2p0AAAAAAAJCrQAAAAAAANq1AAAAAAAAmsUAORztY7TJyQJp+zNxvCHFAgZGzmi3jZkAnOiDQtDx4QBYyKcoYR35AjRXkUO3QgkBUIdyYtU91QA==","dtype":"float64","order":"little","shape":[14]},"xoffsets":[["2010","gross_rent"],["2011","gross_rent"],["2012","gross_rent"],["2013","gross_rent"],["2014","gross_rent"],["2015","gross_rent"],["2016","gross_rent"],["2010","sale_price_sqr_foot"],["2011","sale_price_sqr_foot"],["2012","sale_price_sqr_foot"],["2013","sale_price_sqr_foot"],["2014","sale_price_sqr_foot"],["2015","sale_price_sqr_foot"],["2016","sale_price_sqr_foot"]],"year":["2010","2011","2012","2013","2014","2015","2016","2010","2011","2012","2013","2014","2015","2016"]},"selected":{"id":"8999"},"selection_policy":{"id":"9020"}},"id":"8998","type":"ColumnDataSource"}],"root_ids":["8819","9113"]},"title":"Bokeh Application","version":"2.2.3"}};
    var render_items = [{"docid":"f8ac257a-60a0-433c-a63e-a56a7ea3b5d6","root_ids":["8819"],"roots":{"8819":"9a60e7af-b7e6-4de2-abe2-e6fd0a9cbc6e"}}];
    root.Bokeh.embed.embed_items_notebook(docs_json, render_items);
  }
  if (root.Bokeh !== undefined && root.Bokeh.Panel !== undefined && root['Plotly'] !== undefined ) {
    embed_document(root);
  } else {
    var attempts = 0;
    var timer = setInterval(function(root) {
      if (root.Bokeh !== undefined && root.Bokeh.Panel !== undefined && root['Plotly'] !== undefined) {
        clearInterval(timer);
        embed_document(root);
      } else if (document.readyState == "complete") {
        attempts++;
        if (attempts > 100) {
          clearInterval(timer);
          console.log("Bokeh: ERROR: Unable to run BokehJS code because BokehJS library is missing");
        }
      }
    }, 10, root)
  }
})(window);</script>



# Debugging

Note: Some of the Plotly express plots may not render in the notebook through the panel functions.

However, you can test each plot by uncommenting the following code


```python
housing_units_per_year()
```






<div id='4952'>





  <div class="bk-root" id="1d619142-b586-4137-811d-639a38ecefc8" data-root-id="4952"></div>
</div>
<script type="application/javascript">(function(root) {
  function embed_document(root) {
    var docs_json = {"ff8afc67-12bb-4d48-97bb-f2c11071c3fe":{"roots":{"references":[{"attributes":{"client_comm_id":"aa36d4af48ac4da695e34e284134d6bd","comm_id":"3f1c5e745f5341f0a1a32b9e4f9dac78","plot_id":"4952"},"id":"4953","type":"panel.models.comm_manager.CommManager"},{"attributes":{"height":288,"margin":[5,5,5,5],"name":"Matplotlib07284","text":"&lt;img src=&quot;data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAA2AAAAJACAYAAADrSQUmAAAABHNCSVQICAgIfAhkiAAAAAlwSFlzAAAWJQAAFiUBSVIk8AAAADl0RVh0U29mdHdhcmUAbWF0cGxvdGxpYiB2ZXJzaW9uIDMuMC4zLCBodHRwOi8vbWF0cGxvdGxpYi5vcmcvnQurowAADH9JREFUeJzt1zEBwCAQwMBS/54fFYSBOwVZs2ZmPgAAAI77bwcAAAC8woABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQMSAAQAARAwYAABAxIABAABEDBgAAEDEgAEAAEQMGAAAQGQD+wUIfMggoFoAAAAASUVORK5CYII=&quot; width=&quot;432px&quot; height=&quot;288px&quot; alt=&quot;&quot;&gt;&lt;/img&gt;","width":432},"id":"4952","type":"panel.models.markup.HTML"}],"root_ids":["4952","4953"]},"title":"Bokeh Application","version":"2.2.3"}};
    var render_items = [{"docid":"ff8afc67-12bb-4d48-97bb-f2c11071c3fe","root_ids":["4952"],"roots":{"4952":"1d619142-b586-4137-811d-639a38ecefc8"}}];
    root.Bokeh.embed.embed_items_notebook(docs_json, render_items);
  }
  if (root.Bokeh !== undefined && root.Bokeh.Panel !== undefined && root['Plotly'] !== undefined ) {
    embed_document(root);
  } else {
    var attempts = 0;
    var timer = setInterval(function(root) {
      if (root.Bokeh !== undefined && root.Bokeh.Panel !== undefined && root['Plotly'] !== undefined) {
        clearInterval(timer);
        embed_document(root);
      } else if (document.readyState == "complete") {
        attempts++;
        if (attempts > 100) {
          clearInterval(timer);
          console.log("Bokeh: ERROR: Unable to run BokehJS code because BokehJS library is missing");
        }
      }
    }, 10, root)
  }
})(window);</script>




    
![png](output_files/output_16_2.png)
    



```python
average_gross_rent()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7f9e71ebf090>




    
![png](output_files/output_17_1.png)
    



```python
average_sales_price()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7f9df10fd390>




    
![png](output_files/output_18_1.png)
    



```python
average_price_by_neighborhood()
```






<div id='3842'>





  <div class="bk-root" id="250320d4-25d8-4cff-a700-67d6344f2a15" data-root-id="3842"></div>
</div>
<script type="application/javascript">(function(root) {
  function embed_document(root) {
    var docs_json = {"01777f37-b23f-4762-8540-09e6eaad451e":{"roots":{"references":[{"attributes":{"children":[{"id":"3843"},{"id":"3847"},{"id":"3909"},{"id":"3910"}],"margin":[0,0,0,0],"name":"Row05475"},"id":"3842","type":"Row"},{"attributes":{},"id":"3864","type":"SaveTool"},{"attributes":{"line_alpha":0.1,"line_color":"#30a2da","line_width":2,"x":{"field":"year"},"y":{"field":"sale_price_sqr_foot"}},"id":"3881","type":"Line"},{"attributes":{"line_alpha":0.2,"line_color":"#30a2da","line_width":2,"x":{"field":"year"},"y":{"field":"sale_price_sqr_foot"}},"id":"3882","type":"Line"},{"attributes":{"axis_label":"Year","bounds":"auto","formatter":{"id":"3885"},"major_label_orientation":"horizontal","ticker":{"id":"3857"}},"id":"3856","type":"LinearAxis"},{"attributes":{},"id":"3865","type":"PanTool"},{"attributes":{"overlay":{"id":"3869"}},"id":"3867","type":"BoxZoomTool"},{"attributes":{"bottom_units":"screen","fill_alpha":0.5,"fill_color":"lightgrey","left_units":"screen","level":"overlay","line_alpha":1.0,"line_color":"black","line_dash":[4,4],"line_width":2,"right_units":"screen","top_units":"screen"},"id":"3869","type":"BoxAnnotation"},{"attributes":{"below":[{"id":"3856"}],"center":[{"id":"3859"},{"id":"3863"}],"left":[{"id":"3860"}],"margin":[5,5,5,5],"min_border_bottom":10,"min_border_left":10,"min_border_right":10,"min_border_top":10,"plot_height":300,"plot_width":700,"renderers":[{"id":"3883"}],"sizing_mode":"fixed","title":{"id":"3848"},"toolbar":{"id":"3870"},"x_range":{"id":"3844"},"x_scale":{"id":"3852"},"y_range":{"id":"3845"},"y_scale":{"id":"3854"}},"id":"3847","subtype":"Figure","type":"Plot"},{"attributes":{},"id":"3885","type":"BasicTickFormatter"},{"attributes":{"line_color":"#30a2da","line_width":2,"x":{"field":"year"},"y":{"field":"sale_price_sqr_foot"}},"id":"3880","type":"Line"},{"attributes":{"text":"neighborhood: Alamo Square","text_color":{"value":"black"},"text_font_size":{"value":"12pt"}},"id":"3848","type":"Title"},{"attributes":{"data":{"sale_price_sqr_foot":{"__ndarray__":"Dkc7WO0yckCafszcbwhxQIGRs5ot42ZAJzog0LQ8eEAWMinKGEd+QI0V5FDt0IJAVCHcmLVPdUA=","dtype":"float64","order":"little","shape":[7]},"year":[2010,2011,2012,2013,2014,2015,2016]},"selected":{"id":"3878"},"selection_policy":{"id":"3899"}},"id":"3877","type":"ColumnDataSource"},{"attributes":{"margin":[5,5,5,5],"name":"HSpacer05484","sizing_mode":"stretch_width"},"id":"3843","type":"Spacer"},{"attributes":{},"id":"3866","type":"WheelZoomTool"},{"attributes":{},"id":"3878","type":"Selection"},{"attributes":{"active_drag":"auto","active_inspect":"auto","active_multi":null,"active_scroll":"auto","active_tap":"auto","tools":[{"id":"3846"},{"id":"3864"},{"id":"3865"},{"id":"3866"},{"id":"3867"},{"id":"3868"}]},"id":"3870","type":"Toolbar"},{"attributes":{},"id":"3857","type":"BasicTicker"},{"attributes":{},"id":"3854","type":"LinearScale"},{"attributes":{},"id":"3868","type":"ResetTool"},{"attributes":{"source":{"id":"3877"}},"id":"3884","type":"CDSView"},{"attributes":{},"id":"3861","type":"BasicTicker"},{"attributes":{"margin":[5,5,5,5],"name":"HSpacer05485","sizing_mode":"stretch_width"},"id":"3909","type":"Spacer"},{"attributes":{"axis_label":"Average Sale Price per Square Foot","bounds":"auto","formatter":{"id":"3887"},"major_label_orientation":"horizontal","ticker":{"id":"3861"}},"id":"3860","type":"LinearAxis"},{"attributes":{"callback":null,"renderers":[{"id":"3883"}],"tags":["hv_created"],"tooltips":[["year","@{year}"],["sale_price_sqr_foot","@{sale_price_sqr_foot}"]]},"id":"3846","type":"HoverTool"},{"attributes":{"client_comm_id":"a8f76f28f5b84e3ba0921f5c7d358177","comm_id":"3010bdb6579740e0a25b33b3eaa62cb8","plot_id":"3842"},"id":"3939","type":"panel.models.comm_manager.CommManager"},{"attributes":{"axis":{"id":"3860"},"dimension":1,"grid_line_color":null,"ticker":null},"id":"3863","type":"Grid"},{"attributes":{"margin":[5,5,5,5],"name":"VSpacer05482","sizing_mode":"stretch_height"},"id":"3914","type":"Spacer"},{"attributes":{"data_source":{"id":"3877"},"glyph":{"id":"3880"},"hover_glyph":null,"muted_glyph":{"id":"3882"},"nonselection_glyph":{"id":"3881"},"selection_glyph":null,"view":{"id":"3884"}},"id":"3883","type":"GlyphRenderer"},{"attributes":{},"id":"3852","type":"LinearScale"},{"attributes":{"end":644.0175329447045,"reset_end":644.0175329447045,"reset_start":141.1976609302527,"start":141.1976609302527,"tags":[[["sale_price_sqr_foot","sale_price_sqr_foot",null]]]},"id":"3845","type":"Range1d"},{"attributes":{"axis":{"id":"3856"},"grid_line_color":null,"ticker":null},"id":"3859","type":"Grid"},{"attributes":{"children":[{"id":"3913"}],"css_classes":["panel-widget-box"],"margin":[5,5,5,5],"name":"WidgetBox05476"},"id":"3912","type":"Column"},{"attributes":{"margin":[20,20,20,20],"min_width":250,"options":["Alamo Square","Anza Vista","Bayview","Buena Vista Park","Central Richmond","Central Sunset","Corona Heights","Cow Hollow","Croker Amazon","Diamond Heights","Downtown ","Eureka Valley/Dolores Heights","Excelsior","Financial District North","Financial District South","Forest Knolls","Glen Park","Golden Gate Heights","Haight Ashbury","Hayes Valley","Hunters Point","Ingleside ","Inner Mission","Inner Parkside","Inner Richmond","Inner Sunset","Jordan Park/Laurel Heights","Lake --The Presidio","Lone Mountain","Lower Pacific Heights","Marina","Miraloma Park","Mission Bay","Mission Dolores","Mission Terrace","Nob Hill","Noe Valley","Oceanview","Outer Parkside","Outer Richmond ","Outer Sunset","Pacific Heights","Park North","Parkside","Parnassus/Ashbury Heights","Portola","Potrero Hill","Presidio Heights","Russian Hill","South Beach","South of Market","Sunnyside","Telegraph Hill","Twin Peaks","Union Square District","Van Ness/ Civic Center","West Portal","Western Addition","Yerba Buena","Bernal Heights ","Clarendon Heights","Duboce Triangle","Ingleside Heights","North Beach","North Waterfront","Outer Mission","Westwood Highlands","Merced Heights","Midtown Terrace","Visitacion Valley","Silver Terrace","Westwood Park","Bayview Heights"],"title":"neighborhood","value":"Alamo Square","width":250},"id":"3913","type":"Select"},{"attributes":{"end":2016.0,"reset_end":2016.0,"reset_start":2010.0,"start":2010.0,"tags":[[["year","year",null]]]},"id":"3844","type":"Range1d"},{"attributes":{"margin":[5,5,5,5],"name":"VSpacer05481","sizing_mode":"stretch_height"},"id":"3911","type":"Spacer"},{"attributes":{},"id":"3899","type":"UnionRenderers"},{"attributes":{},"id":"3887","type":"BasicTickFormatter"},{"attributes":{"children":[{"id":"3911"},{"id":"3912"},{"id":"3914"}],"margin":[0,0,0,0],"name":"Column05483"},"id":"3910","type":"Column"}],"root_ids":["3842","3939"]},"title":"Bokeh Application","version":"2.2.3"}};
    var render_items = [{"docid":"01777f37-b23f-4762-8540-09e6eaad451e","root_ids":["3842"],"roots":{"3842":"250320d4-25d8-4cff-a700-67d6344f2a15"}}];
    root.Bokeh.embed.embed_items_notebook(docs_json, render_items);
  }
  if (root.Bokeh !== undefined && root.Bokeh.Panel !== undefined && root['Plotly'] !== undefined ) {
    embed_document(root);
  } else {
    var attempts = 0;
    var timer = setInterval(function(root) {
      if (root.Bokeh !== undefined && root.Bokeh.Panel !== undefined && root['Plotly'] !== undefined) {
        clearInterval(timer);
        embed_document(root);
      } else if (document.readyState == "complete") {
        attempts++;
        if (attempts > 100) {
          clearInterval(timer);
          console.log("Bokeh: ERROR: Unable to run BokehJS code because BokehJS library is missing");
        }
      }
    }, 10, root)
  }
})(window);</script>




```python
top_most_expensive_neighborhoods()
```






<div id='2324'>





  <div class="bk-root" id="b7a35076-392e-4789-8ca1-5fda08c05fef" data-root-id="2324"></div>
</div>
<script type="application/javascript">(function(root) {
  function embed_document(root) {
    var docs_json = {"ea3e4325-032c-4f15-b06e-4117259af2b4":{"roots":{"references":[{"attributes":{"axis":{"id":"2341"},"dimension":1,"grid_line_color":null,"ticker":null},"id":"2344","type":"Grid"},{"attributes":{},"id":"2339","type":"CategoricalTicker"},{"attributes":{"axis":{"id":"2338"},"grid_line_color":null,"ticker":null},"id":"2340","type":"Grid"},{"attributes":{"margin":[5,5,5,5],"name":"HSpacer03488","sizing_mode":"stretch_width"},"id":"2325","type":"Spacer"},{"attributes":{"factors":["Union Square District","Merced Heights","Miraloma Park","Pacific Heights","Westwood Park","Telegraph Hill","Presidio Heights","Cow Hollow","Potrero Hill","South Beach"],"tags":[[["neighborhood","neighborhood",null]]]},"id":"2326","type":"FactorRange"},{"attributes":{},"id":"2342","type":"BasicTicker"},{"attributes":{"axis_label":"Neighborhood","bounds":"auto","formatter":{"id":"2367"},"major_label_orientation":0.7853981633974483,"ticker":{"id":"2339"}},"id":"2338","type":"CategoricalAxis"},{"attributes":{"axis_label":"Average Sale Price per Square Foot","bounds":"auto","formatter":{"id":"2368"},"major_label_orientation":"horizontal","ticker":{"id":"2342"}},"id":"2341","type":"LinearAxis"},{"attributes":{"fill_color":{"value":"#30a2da"},"top":{"field":"sale_price_sqr_foot"},"width":{"value":0.8},"x":{"field":"neighborhood"}},"id":"2361","type":"VBar"},{"attributes":{},"id":"2334","type":"CategoricalScale"},{"attributes":{"data_source":{"id":"2358"},"glyph":{"id":"2361"},"hover_glyph":null,"muted_glyph":{"id":"2363"},"nonselection_glyph":{"id":"2362"},"selection_glyph":null,"view":{"id":"2365"}},"id":"2364","type":"GlyphRenderer"},{"attributes":{"bottom_units":"screen","fill_alpha":0.5,"fill_color":"lightgrey","left_units":"screen","level":"overlay","line_alpha":1.0,"line_color":"black","line_dash":[4,4],"line_width":2,"right_units":"screen","top_units":"screen"},"id":"2350","type":"BoxAnnotation"},{"attributes":{"end":929.3801355198136,"reset_end":929.3801355198136,"reset_start":0.0,"tags":[[["sale_price_sqr_foot","sale_price_sqr_foot",null]]]},"id":"2327","type":"Range1d"},{"attributes":{"source":{"id":"2358"}},"id":"2365","type":"CDSView"},{"attributes":{},"id":"2376","type":"UnionRenderers"},{"attributes":{},"id":"2359","type":"Selection"},{"attributes":{},"id":"2367","type":"CategoricalTickFormatter"},{"attributes":{"active_drag":"auto","active_inspect":"auto","active_multi":null,"active_scroll":"auto","active_tap":"auto","tools":[{"id":"2328"},{"id":"2345"},{"id":"2346"},{"id":"2347"},{"id":"2348"},{"id":"2349"}]},"id":"2351","type":"Toolbar"},{"attributes":{"data":{"neighborhood":["Union Square District","Merced Heights","Miraloma Park","Pacific Heights","Westwood Park","Telegraph Hill","Presidio Heights","Cow Hollow","Potrero Hill","South Beach"],"sale_price_sqr_foot":{"__ndarray__":"JRAUMfI/jEB9IrkvwqaIQFx/3pp8XohAFpsVUHKMhUBs0ERas3iFQL0KhXgNJIVAy835O80ahUANV25bts+EQJrMCuEbsIRAya3y7v5QhEA=","dtype":"float64","order":"little","shape":[10]}},"selected":{"id":"2359"},"selection_policy":{"id":"2376"}},"id":"2358","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"value":0.2},"fill_color":{"value":"#30a2da"},"line_alpha":{"value":0.2},"top":{"field":"sale_price_sqr_foot"},"width":{"value":0.8},"x":{"field":"neighborhood"}},"id":"2363","type":"VBar"},{"attributes":{},"id":"2368","type":"BasicTickFormatter"},{"attributes":{},"id":"2345","type":"SaveTool"},{"attributes":{},"id":"2346","type":"PanTool"},{"attributes":{},"id":"2347","type":"WheelZoomTool"},{"attributes":{"children":[{"id":"2325"},{"id":"2329"},{"id":"2386"}],"margin":[0,0,0,0],"name":"Row03484","tags":["embedded"]},"id":"2324","type":"Row"},{"attributes":{"below":[{"id":"2338"}],"center":[{"id":"2340"},{"id":"2344"}],"left":[{"id":"2341"}],"margin":[5,5,5,5],"min_border_bottom":10,"min_border_left":10,"min_border_right":10,"min_border_top":10,"plot_height":400,"plot_width":700,"renderers":[{"id":"2364"}],"sizing_mode":"fixed","title":{"id":"2330"},"toolbar":{"id":"2351"},"x_range":{"id":"2326"},"x_scale":{"id":"2334"},"y_range":{"id":"2327"},"y_scale":{"id":"2336"}},"id":"2329","subtype":"Figure","type":"Plot"},{"attributes":{"overlay":{"id":"2350"}},"id":"2348","type":"BoxZoomTool"},{"attributes":{},"id":"2336","type":"LinearScale"},{"attributes":{"fill_alpha":{"value":0.1},"fill_color":{"value":"#30a2da"},"line_alpha":{"value":0.1},"top":{"field":"sale_price_sqr_foot"},"width":{"value":0.8},"x":{"field":"neighborhood"}},"id":"2362","type":"VBar"},{"attributes":{"margin":[5,5,5,5],"name":"HSpacer03489","sizing_mode":"stretch_width"},"id":"2386","type":"Spacer"},{"attributes":{"callback":null,"renderers":[{"id":"2364"}],"tags":["hv_created"],"tooltips":[["neighborhood","@{neighborhood}"],["sale_price_sqr_foot","@{sale_price_sqr_foot}"]]},"id":"2328","type":"HoverTool"},{"attributes":{},"id":"2349","type":"ResetTool"},{"attributes":{"text":"Top 10 Most Expensive Neighborhoods in SFO","text_color":{"value":"black"},"text_font_size":{"value":"12pt"}},"id":"2330","type":"Title"}],"root_ids":["2324"]},"title":"Bokeh Application","version":"2.2.3"}};
    var render_items = [{"docid":"ea3e4325-032c-4f15-b06e-4117259af2b4","root_ids":["2324"],"roots":{"2324":"b7a35076-392e-4789-8ca1-5fda08c05fef"}}];
    root.Bokeh.embed.embed_items_notebook(docs_json, render_items);
  }
  if (root.Bokeh !== undefined && root.Bokeh.Panel !== undefined && root['Plotly'] !== undefined ) {
    embed_document(root);
  } else {
    var attempts = 0;
    var timer = setInterval(function(root) {
      if (root.Bokeh !== undefined && root.Bokeh.Panel !== undefined && root['Plotly'] !== undefined) {
        clearInterval(timer);
        embed_document(root);
      } else if (document.readyState == "complete") {
        attempts++;
        if (attempts > 100) {
          clearInterval(timer);
          console.log("Bokeh: ERROR: Unable to run BokehJS code because BokehJS library is missing");
        }
      }
    }, 10, root)
  }
})(window);</script>




```python
most_expensive_neighborhoods_rent_sales()
```






<div id='2428'>





  <div class="bk-root" id="4812a090-431b-4dff-8c0a-a0106107daaa" data-root-id="2428"></div>
</div>
<script type="application/javascript">(function(root) {
  function embed_document(root) {
    var docs_json = {"83978e91-aba4-4211-ab27-566d2f884115":{"roots":{"references":[{"attributes":{"margin":[5,5,5,5],"name":"HSpacer03593","sizing_mode":"stretch_width"},"id":"2495","type":"Spacer"},{"attributes":{"children":[{"id":"2497"},{"id":"2498"},{"id":"2500"}],"margin":[0,0,0,0],"name":"Column03591"},"id":"2496","type":"Column"},{"attributes":{},"id":"2438","type":"CategoricalScale"},{"attributes":{"factors":[["2010","gross_rent"],["2010","sale_price_sqr_foot"],["2011","gross_rent"],["2011","sale_price_sqr_foot"],["2012","gross_rent"],["2012","sale_price_sqr_foot"],["2013","gross_rent"],["2013","sale_price_sqr_foot"],["2014","gross_rent"],["2014","sale_price_sqr_foot"],["2015","gross_rent"],["2015","sale_price_sqr_foot"],["2016","gross_rent"],["2016","sale_price_sqr_foot"]],"tags":[[["year","year",null],["Variable","Variable",null]]]},"id":"2430","type":"FactorRange"},{"attributes":{"factors":["gross_rent","sale_price_sqr_foot"],"palette":["#30a2da","#fc4f30"]},"id":"2462","type":"CategoricalColorMapper"},{"attributes":{"text":"Top 10 Most Expensive Neighborhoods in SFO","text_color":{"value":"black"},"text_font_size":{"value":"12pt"}},"id":"2434","type":"Title"},{"attributes":{},"id":"2472","type":"CategoricalTickFormatter"},{"attributes":{"children":[{"id":"2429"},{"id":"2433"},{"id":"2495"},{"id":"2496"}],"margin":[0,0,0,0],"name":"Row03583"},"id":"2428","type":"Row"},{"attributes":{},"id":"2443","type":"CategoricalTicker"},{"attributes":{"margin":[5,5,5,5],"name":"VSpacer03589","sizing_mode":"stretch_height"},"id":"2497","type":"Spacer"},{"attributes":{"data":{"Variable":["gross_rent","gross_rent","gross_rent","gross_rent","gross_rent","gross_rent","gross_rent","sale_price_sqr_foot","sale_price_sqr_foot","sale_price_sqr_foot","sale_price_sqr_foot","sale_price_sqr_foot","sale_price_sqr_foot","sale_price_sqr_foot"],"value":{"__ndarray__":"AAAAAABck0AAAAAAAOiXQAAAAAAAKKJAAAAAAAA2p0AAAAAAAJCrQAAAAAAANq1AAAAAAAAmsUAORztY7TJyQJp+zNxvCHFAgZGzmi3jZkAnOiDQtDx4QBYyKcoYR35AjRXkUO3QgkBUIdyYtU91QA==","dtype":"float64","order":"little","shape":[14]},"xoffsets":[["2010","gross_rent"],["2011","gross_rent"],["2012","gross_rent"],["2013","gross_rent"],["2014","gross_rent"],["2015","gross_rent"],["2016","gross_rent"],["2010","sale_price_sqr_foot"],["2011","sale_price_sqr_foot"],["2012","sale_price_sqr_foot"],["2013","sale_price_sqr_foot"],["2014","sale_price_sqr_foot"],["2015","sale_price_sqr_foot"],["2016","sale_price_sqr_foot"]],"year":["2010","2011","2012","2013","2014","2015","2016","2010","2011","2012","2013","2014","2015","2016"]},"selected":{"id":"2464"},"selection_policy":{"id":"2485"}},"id":"2463","type":"ColumnDataSource"},{"attributes":{"axis_label":"Neighborhood","bounds":"auto","formatter":{"id":"2472"},"major_label_orientation":0.7853981633974483,"ticker":{"id":"2443"}},"id":"2442","type":"CategoricalAxis"},{"attributes":{"axis":{"id":"2445"},"dimension":1,"grid_line_color":null,"ticker":null},"id":"2448","type":"Grid"},{"attributes":{},"id":"2464","type":"Selection"},{"attributes":{"axis":{"id":"2442"},"grid_line_color":null,"ticker":null},"id":"2444","type":"Grid"},{"attributes":{"children":[{"id":"2499"}],"css_classes":["panel-widget-box"],"margin":[5,5,5,5],"name":"WidgetBox03584"},"id":"2498","type":"Column"},{"attributes":{},"id":"2485","type":"UnionRenderers"},{"attributes":{},"id":"2473","type":"BasicTickFormatter"},{"attributes":{},"id":"2446","type":"BasicTicker"},{"attributes":{"margin":[5,5,5,5],"name":"HSpacer03592","sizing_mode":"stretch_width"},"id":"2429","type":"Spacer"},{"attributes":{"axis_label":"Num Housing Units","bounds":"auto","formatter":{"id":"2473"},"major_label_orientation":"horizontal","ticker":{"id":"2446"}},"id":"2445","type":"LinearAxis"},{"attributes":{},"id":"2440","type":"LinearScale"},{"attributes":{"margin":[5,5,5,5],"name":"VSpacer03590","sizing_mode":"stretch_height"},"id":"2500","type":"Spacer"},{"attributes":{"end":4810.690068306854,"reset_end":4810.690068306854,"reset_start":0.0,"tags":[[["value","value",null]]]},"id":"2431","type":"Range1d"},{"attributes":{"bottom_units":"screen","fill_alpha":0.5,"fill_color":"lightgrey","left_units":"screen","level":"overlay","line_alpha":1.0,"line_color":"black","line_dash":[4,4],"line_width":2,"right_units":"screen","top_units":"screen"},"id":"2454","type":"BoxAnnotation"},{"attributes":{"fill_alpha":{"value":0.1},"fill_color":{"field":"Variable","transform":{"id":"2462"}},"line_alpha":{"value":0.1},"top":{"field":"value"},"width":{"value":0.8},"x":{"field":"xoffsets"}},"id":"2467","type":"VBar"},{"attributes":{"margin":[20,20,20,20],"min_width":250,"options":["Alamo Square","Anza Vista","Bayview","Buena Vista Park","Central Richmond","Central Sunset","Corona Heights","Cow Hollow","Croker Amazon","Diamond Heights","Downtown ","Eureka Valley/Dolores Heights","Excelsior","Financial District North","Financial District South","Forest Knolls","Glen Park","Golden Gate Heights","Haight Ashbury","Hayes Valley","Hunters Point","Ingleside ","Inner Mission","Inner Parkside","Inner Richmond","Inner Sunset","Jordan Park/Laurel Heights","Lake --The Presidio","Lone Mountain","Lower Pacific Heights","Marina","Miraloma Park","Mission Bay","Mission Dolores","Mission Terrace","Nob Hill","Noe Valley","Oceanview","Outer Parkside","Outer Richmond ","Outer Sunset","Pacific Heights","Park North","Parkside","Parnassus/Ashbury Heights","Portola","Potrero Hill","Presidio Heights","Russian Hill","South Beach","South of Market","Sunnyside","Telegraph Hill","Twin Peaks","Union Square District","Van Ness/ Civic Center","West Portal","Western Addition","Yerba Buena","Bernal Heights ","Clarendon Heights","Duboce Triangle","Ingleside Heights","North Beach","North Waterfront","Outer Mission","Westwood Highlands","Merced Heights","Midtown Terrace","Visitacion Valley","Silver Terrace","Westwood Park","Bayview Heights"],"title":"neighborhood","value":"Alamo Square","width":250},"id":"2499","type":"Select"},{"attributes":{"active_drag":"auto","active_inspect":"auto","active_multi":null,"active_scroll":"auto","active_tap":"auto","tools":[{"id":"2432"},{"id":"2449"},{"id":"2450"},{"id":"2451"},{"id":"2452"},{"id":"2453"}]},"id":"2455","type":"Toolbar"},{"attributes":{"source":{"id":"2463"}},"id":"2470","type":"CDSView"},{"attributes":{},"id":"2449","type":"SaveTool"},{"attributes":{"data_source":{"id":"2463"},"glyph":{"id":"2466"},"hover_glyph":null,"muted_glyph":{"id":"2468"},"nonselection_glyph":{"id":"2467"},"selection_glyph":null,"view":{"id":"2470"}},"id":"2469","type":"GlyphRenderer"},{"attributes":{"below":[{"id":"2442"}],"center":[{"id":"2444"},{"id":"2448"}],"left":[{"id":"2445"}],"margin":[5,5,5,5],"min_border_bottom":10,"min_border_left":10,"min_border_right":10,"min_border_top":10,"plot_height":400,"plot_width":700,"renderers":[{"id":"2469"}],"sizing_mode":"fixed","title":{"id":"2434"},"toolbar":{"id":"2455"},"x_range":{"id":"2430"},"x_scale":{"id":"2438"},"y_range":{"id":"2431"},"y_scale":{"id":"2440"}},"id":"2433","subtype":"Figure","type":"Plot"},{"attributes":{"callback":null,"renderers":[{"id":"2469"}],"tags":["hv_created"],"tooltips":[["year","@{year}"],["Variable","@{Variable}"],["value","@{value}"]]},"id":"2432","type":"HoverTool"},{"attributes":{},"id":"2450","type":"PanTool"},{"attributes":{"fill_alpha":{"value":0.2},"fill_color":{"field":"Variable","transform":{"id":"2462"}},"line_alpha":{"value":0.2},"top":{"field":"value"},"width":{"value":0.8},"x":{"field":"xoffsets"}},"id":"2468","type":"VBar"},{"attributes":{},"id":"2451","type":"WheelZoomTool"},{"attributes":{},"id":"2453","type":"ResetTool"},{"attributes":{"fill_color":{"field":"Variable","transform":{"id":"2462"}},"top":{"field":"value"},"width":{"value":0.8},"x":{"field":"xoffsets"}},"id":"2466","type":"VBar"},{"attributes":{"overlay":{"id":"2454"}},"id":"2452","type":"BoxZoomTool"},{"attributes":{"client_comm_id":"650971bc280a4decba21368bf9fbddd9","comm_id":"542c8c3e547c4c26a6a6c0a2e5622532","plot_id":"2428"},"id":"2525","type":"panel.models.comm_manager.CommManager"}],"root_ids":["2428","2525"]},"title":"Bokeh Application","version":"2.2.3"}};
    var render_items = [{"docid":"83978e91-aba4-4211-ab27-566d2f884115","root_ids":["2428"],"roots":{"2428":"4812a090-431b-4dff-8c0a-a0106107daaa"}}];
    root.Bokeh.embed.embed_items_notebook(docs_json, render_items);
  }
  if (root.Bokeh !== undefined && root.Bokeh.Panel !== undefined && root['Plotly'] !== undefined ) {
    embed_document(root);
  } else {
    var attempts = 0;
    var timer = setInterval(function(root) {
      if (root.Bokeh !== undefined && root.Bokeh.Panel !== undefined && root['Plotly'] !== undefined) {
        clearInterval(timer);
        embed_document(root);
      } else if (document.readyState == "complete") {
        attempts++;
        if (attempts > 100) {
          clearInterval(timer);
          console.log("Bokeh: ERROR: Unable to run BokehJS code because BokehJS library is missing");
        }
      }
    }, 10, root)
  }
})(window);</script>




```python
neighborhood_map().show()
```

    Launching server at http://localhost:62411





    <bokeh.server.server.Server at 0x7f9e67913490>




```python
# parallel_categories()
```


```python
# parallel_coordinates()
```


```python
# sunburst()
```


```python

```
