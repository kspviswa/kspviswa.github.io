---
layout: post
title: Realizing Gaussian law of randomness
categories: musings, Maths
tags:
- Maths
- Musings
---

During our school / college days, we would have studied random distributions, probability, mean etc. We would have also studied the famous **Gauss Law** or more famously known for **Bell Curve**.

In simple words, *Gauss Law* predicts that ( it proved way back though!!) any randomness has a pattern and the pattern looks like a **bell curve**.


 ![Image of BellCurve](http://hyperphysics.phy-astr.gsu.edu/hbase/math/immath/gauds.gif)


This act of randomness is true irrespective of the values / domains on which this randomness prevail. Hence this *bell curve* distribution has been used as a measurement tool in variety of domains like errors / noises and even in corporate appraisal cycles to rank employees.

Though this information is well known / well tutored right from our class X, I didn't realize the truth behind this, until I saw a [Ted video](https://www.ted.com/talks/cedric_villani_what_s_so_sexy_about_math) about Mathematics from [Cédric Villani](https://en.wikipedia.org/wiki/C%C3%A9dric_Villani), where he explains about the applications of mathematics and it's pure abstract nature.

In the talk, he demonstrated how *Gauss Law* a.k.a *Law of errors* can be applied to anything, by using a simple device which transports tiny beads from top to bottom via various random paths. Each bead take a random path to reach the bottom and that pattern always follow this *bell curve*.


## My experiment to realize the truth

### Hello World

Quite intrigued by this video, I set out to test the randomness pattern myself. First experiment was to play with the `rand()` function which is available in almost every language. The intent was to generate a number randomly ( not a random prime number ) for a given period and then plot them to see what pattern it is taking.

{% highlight javascript %}
<html>
  <head>
    <script type="text/javascript" src="https://www.gstatic.com/charts/loader.js"></script>
    <script type="text/javascript">
      google.charts.load('current', {'packages':['corechart']});
      google.charts.setOnLoadCallback(drawChart);

	  var gdata;
	  
	  function getRandomArbitrary(min, max) {
		return Math.floor(Math.random() * (max - min) + min);
		}
		
      function drawChart() {
        var data = google.visualization.arrayToDataTable([
          ['Year', 'Sales', 'Expenses'],
          ['2004',  getRandomArbitrary(1,1000), getRandomArbitrary(1,1000)],
          ['2005',  getRandomArbitrary(1,1000), getRandomArbitrary(1,1000)],
          ['2006',  getRandomArbitrary(1,1000), getRandomArbitrary(1,1000)],
          ['2007',  getRandomArbitrary(1,1000), getRandomArbitrary(1,1000)],
		  ['2008', getRandomArbitrary(1,1000), getRandomArbitrary(1,1000)]
        ]);
		

        var options = {
          title: 'Ramdom distribution experiment',
          curveType: 'function',
          legend: { position: 'bottom' }
        };

        var chart = new google.visualization.LineChart(document.getElementById('curve_chart'));

        chart.draw(data, options);
		});
      }
    </script>
  </head>
  <body>
    <div id="curve_chart" style="width: 900px; height: 500px"></div>
  </body>
</html>
{% endhighlight %}

You can see a live preview of the chart, refreshed at regular intervals below. As you can see, the randomness in this experiment, **do always follow the Gauss Law**

<iframe src="{{ site.baseurl }}static/iframe/hello_world.html" width="1000" height="500"></iframe>

### Actual World

As you can probably guess, above experiment was very brute-force. The `rand()` has been engineered to follow the `bell curve` i.e `normal distribution`/ Hence there is no surprise in the output. It did not really help to prove the fact that, `Gauss Law` is universal. 

However, using this code as base, I tried to get the World population of countries in random way. Luckily there exists a [API available](https://restcountries.eu/) to retrieve the information about every country in the world. Using this, I tried to query the population of a random country and tried to plot that in the graph.

Remember this act is **truely random** i.e **I never know the population of each country before in hand + the choice of the country is not pre-decided. Everything changes at the start of every sample of experiment**. 

Below *iFrame* would show you the live preview of the graph.  As you can see, the randomness in this experiment, **do always follow the Gauss Law**.

<iframe src="{{ site.baseurl }}static/iframe/real_world.html" width="1000" height="500"></iframe>

## Math is Natural Science

This once property of uniform abstraction in mathematics, really amazes me. As rightly put by Dr [Cédric Villani](https://en.wikipedia.org/wiki/C%C3%A9dric_Villani) in his talk, 

> "Mathematics allows us to go beyond the intuition and explore territories which do not fit within our grasp.".

Cya :-)


