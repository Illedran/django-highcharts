## Django Highcharts

This project is inspired from a fork of novapost package to
generate charts in your Django application using Highcharts helpers.

- Pie with drilldown charts
- 3D Pie Options
- Speedometer charts
- Multiple Axes charts
- Area charts
- Bar charts
- Heatmap charts
- Polar Spider web charts
- HighStock basic charts

* `Source code is on Github <https://github.com/ernestoarbitrio/django-highcharts>`

### Requirements
 - `django > 1.8 (tested with 1.8.x and 1.11.x)`
 - `python 2.7.x` or `python 3.x`

### Install
There are a few different ways you can install pyechonest:

* Use setuptools: `pip install django-highcharts`
* Download the zipfile from the [downloads](https://github.com/ernestoarbitrio/django-highcharts/archive/master.zip) page and install it. 
* Checkout the source: `git@github.com:ernestoarbitrio/django-highcharts.git` and install it yourself.

In your settings file:
```
INSTALLED_APPS = [
   ### other apps ###
   'highcharts'
   ### other apps ###
]
```

Don’t forget to set your STATIC_ROOT path and to run the following command to update the static files:

`python manage.py collectstatic`

Write a graph with different series type (in view.py file or if you want in a graph.py file):
```python
from highcharts.views import (HighChartsMultiAxesView, HighChartsPieView,
                              HighChartsSpeedometerView, HighChartsHeatMapView, HighChartsPolarView)
                              
class BarView(HighChartsMultiAxesView):
    title = 'Example Bar Chart'
    subtitle = 'my subtitle'
    categories = ['Orange', 'Bananas', 'Apples']
    chart_type = ''
    chart = {'zoomType': 'xy'}
    tooltip = {'shared': 'true'}
    legend = {'layout': 'horizontal', 'align': 'left',
              'floating': 'true', 'verticalAlign': 'top',
              'y': -10, 'borderColor': '#e3e3e3'}

    @property
    def yaxis(self):
        y_axis = [
            {'labels': {'format': '{value} pz/sc ', 'style': {'color': '#f67d0a'}},
             'title': {'text': "Oranges", 'style': {'color': '#f67d0a'}},
             'opposite': 'true'},
            {'gridLineWidth': 1,
             'title': {'text': "Bananas", 'style': {'color': '#3771c8'}},
             'labels': {'style': {'color': '#3771c8'}, 'format': '{value} euro'}},
            {'gridLineWidth': 1,
             'title': {'text': "Apples", 'style': {'color': '#666666'}},
             'labels': {'format': '{value} pz', 'style': {'color': '#666666'}},
             'opposite': 'true'}
        ]
        return y_axis

    @property
    def series(self):
        series = [
            {
                'name': 'Orange',
                'type': 'column',
                'yAxis': 1,
                'data': [90,44,55,67,4,5,6,3,2,45,2,3,2,45,5],
                'tooltip': "{ valueSuffix: ' euro' }",
                'color': '#3771c8'
            },
            {
                'name': 'Bananas',
                'type': 'spline',
                'yAxis': 2,
                'data': [12,34,34,34, 5,34,3,45,2,3,2,4,4,1,23],
                'marker': { 'enabled': 'true' },
                'dashStyle': 'shortdot',
                'color': '#666666',
                },
            {
                'name': 'Apples',
                'type': 'spline',
                'data': [12,23,23,23,21,4,4,76,3,66,6,4,5,2,3],
                'color': '#f67d0a'
            }
        ]
        return series
```
if you want you can write a graph based on a particular class of chart. For example if you need a pie chart with drilldown interaction:
```python
from highcharts.views import (HighChartsMultiAxesView, HighChartsPieView,
                              HighChartsSpeedometerView, HighChartsHeatMapView, HighChartsPolarView)

class PieDrilldown(HighChartsPieView):
    title = 'Torta'
    subtitle = 'torino'

    @property
    def series(self):
        series = [
            {
                'name': 'Classi',
                'colorByPoint': 'true',
                'data': [
                    {'name': 'Emorroidi',
                     'y': 10,
                     'drilldown': 'emorroidi'},
                    {'name': 'Igiene e bellezza',
                     'y': 12,
                     'drilldown': 'igiene'},
                    {'name': 'Omeopatia',
                     'y': 8,
                     'drilldown': 'omeopatia'}
                ]
            }
        ]
        return series

    @property
    def drilldown(self):
        drilldown = {
            'series': [
                {'id': 'emorroidi',
                 'name': 'Emorroidi',
                 'data': [
                     ['brand1', 7],
                     ['brand2', 3],
                     ['brand3', 5]
                 ]},
                {'id': 'igiene',
                 'name': 'Igiene e Bellezza',
                 'data': [
                     ['brand1', 3],
                     ['brand2', 1],
                     ['brand3', 4],
                     ['brand4', 5]
                 ]},
                {'id': 'omeopatia',
                 'name': 'Omeopatia',
                 'data': [
                     ['brand1', 3],
                     ['brand2', 1],
                     ['brand3', 4],
                     ['', 0]
                 ]}
            ]
        }
        return drilldown

       
```


Then you need to map the graph to an url in url.py file:
```
   from graphs import BarView
   url(regex='^bar/$', view=BarView.as_view(), name='bar')
```

In the template:
```html
   {% load highcharts_tags %}
   <script src="https://code.jquery.com/jquery-3.2.1.min.js" integrity="sha256-hwg4gsxgFZhOsEEamdOYGBf13FyQuiTwlAQgxVSNgt4=" crossorigin="anonymous"></script>
   <!-- enable highcharts scripts -->
   <!-- highcharts_js (highcharts 3d highstock heatmap) you need to pass 1 or 0 if you want to enable 3d or highstock         etc...-->
   {% highcharts_js 1 0 0 0 %}
   <!-- the graph container -->
   <div id="container" style="height: 400px; min-width: 310px; max-width: 1200px; margin: 0 auto"></div>
   <!-- the javascript call -->
   $(function () {
       $.getJSON("{% url 'bar' %}", function(data) {
           $('#container').highcharts(data);
       });
   })
```

Naturally you need to write a standard django view to render the HTML that in turn call, via AJAX, the graph url and
render the chart in the related html chart.
For example:
```python
from django.shortcuts import render
from django.views.generic import TemplateView


class PlotView(TemplateView):
    template_name = "plotter/plot.html"
# Create your views here.
```

An advanced example with parameters passed via url and data retrived from db (using orm or raw query)
```python
class AdvancedGraph(HighChartsMultiAxesView):
    title = 'Advanced graph'
    subtitle = 'params and query'
    chart = {'zoomType': 'xy'}
    tooltip = {'shared': 'true'}
    legend = {
        'layout': 'vertical',
        'align': 'left',
        'verticalAlign': 'top',
        'y': 30
    }

    def get_data(self):
        param = self.kwargs['param1']
        f = MyModel.objects.get(field=param)
        cursor = connection.cursor()
        cursor.execute("select * from mydbfunction(%s)" as (outvalues json)", [f.pk])
        graph = cursor.fetchall()
       
        #### SERIES
        self.serie = graph[0]
        
        ##### X LABELS
        self.categories = graph[1]

        ##### Y AXIS DEFINITIONS
        self.yaxis = {
            'title': {
                'text': 'Title 1'
            },
            'plotLines': [
                {
                    'value': 0,
                    'width': 1,
                    'color': '#808080'
                }
            ]
        }

        ##### SERIES WITH VALUES
        self.series = self.serie
        data = super(AdvancedGraph, self).get_data()
        return data
```
then in urls.py
Then you need to map the graph to an url in url.py file:
```python
   from graphs.py import AdvancedGraph
   url(regex='^adv/(?P<param1>\d+)/$', view=AdvancedGraph.as_view(), name='adv')
```
