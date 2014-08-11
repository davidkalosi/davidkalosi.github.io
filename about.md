---
layout: page
permalink: /about/index.html
title: David Kalosi
tags: []
imagefeature: cover1.jpg
chart: true
---

After some three+ decades of beeing around I came to a shocking conclusion that I might actually have something reasonable to share with the world from time to time.
So welcome, my name is

<figure>
  <img src="{{ site.url }}/images/david.jpg" alt="David Kalosi">
  <figcaption>David Kalosi</figcaption>
</figure>

and I am a software architect/entrepreneur/dba/double Oracle Certified Master/ex guitarist/ex studio owner and heavy music producer/father/pitbull lover/chef de amateur/and some more

and I enjoy writing about

<div class="chart" id="chartdiv" style="width: 100%; height: 500px; margin-bottom: 20px;" ></div>

Be creative, embrace change and most importantly **enjoy life**. Then life will enjoy you.

<!-- amCharts javascript code -->
<script type="text/javascript">
  AmCharts.makeChart("chartdiv",
    {
      "type": "pie",
      "pathToImages": "http://cdn.amcharts.com/lib/3/images/",
      "balloonText": "[[title]]<br><span style='font-size:14px'><b>[[value]]</b> ([[percents]]%)</span>",
      "innerRadius": "40%",
      "labelRadius": 10,
      "labelRadiusField": "Not set",
      "startRadius": "10%",
      "colorField": "Not set",
      "colors": ["#EC0033","#B12C49","#990021","#F53D65","#F56E8B"],
      "descriptionField": "Not set",
      "hoverAlpha": 0.75,
      "outlineThickness": 0,
      "startEffect": "elastic",
      "titleField": "category",
      "valueField": "number-of-posts",
      "allLabels": [],
      "balloon": {},
      "titles": [],
      "dataProvider": [
{% assign tags_list = site.categories %}  
  {% if tags_list.first[0] == null %}
    {% for tag in tags_list %} 
        {
          "category": "{{ tag | capitalize }}",
          "number-of-posts": {{ site.tags[tag].size }}
        },
    {% endfor %}
  {% else %}
    {% for tag in tags_list %} 
        {
          "category": "{{ tag[0] | capitalize }}",
          "number-of-posts": {{ tag[1].size }}
        },
    {% endfor %}
  {% endif %}
{% assign tags_list = nil %}
      ]
    }
  );
</script>
