---
layout: post
subtitle: 'Analysis of twitteR ''#peakdistrict'''
date: '2016-04-16 14:18:00 +0800'
published: true
title: Quantifying visitor sentiment for a National Park
---
## Why access twitter? 

Current **customer relationship management (CRM)** systems create customer profiles based on demographics, past buying patterns and other interactions or actions. Monitoring and quantifying interactions of customers with large areas of landscape such as National Parks with regular comprehensive visitor surveying is costly and prohibitively expensive. 

The emergence of both **social media mining (SMM)**, other data science methods and applications have provided us with the tools to access this information. This provides a great opportunity to develop CRM knowledge and monitoring. 

SMM is not without it's limitations. Preprocessing steps are required to remove the noise and error from the data. Due to the clutter, which is present in most unstructured data, removing the noise from data is difficult and not always successful. For the purpose of this analysis, not all tweets will contain sentiment, for example these could be factual or nonsensical.

## How is this done?

Created and app at <https://dev.twitter.com/apps> to access and reqest information from the Twitter
API. The _ROAuth_ package in [R](https://www.r-project.org/) is used to allow the third party app to access the Twitter API.

The analysis then follows these steps:

1. Search and Clean Tweets
2. Estimate Sentiment (_Naive Algorithm_)
3. Estimate Sentiment (_Naive Bayes_)

## Search and Clean Tweets

For the purposes of obtaining sentiment of Twitter users we must gather data based on the term
'#peakdistrict'. 

{% highlight r %}
#search twiter for 4000 tweets #peakdistrict
PDNP_Tweets = searchTwitter("peakdistrict", n = 4000, lang="en")
{% endhighlight %}

Tweets returned are cleaned (4000 observations) based on following parameters (which provide 1126 
observations):

* Meta information usch as @people, URLs and #hashtags
* Punctuation marks, numbers and unnecessary spaces
* retweets (**RT**) not useful for sentiment analysis

## Estimating Sentiment

As mentioned earlier, some Tweets will be , factual, nonesensical or specific such as customer care
responses. This analysis has not attempted to remove tweets from the data. 

### _Naive Algorithm_ 

The Naive algorithm gives a score based on the number of times a positive of negative word occurred
in the given sentence (or Tweet). To do this, the positive and negative _opinion lexicon_ is 
[downloaded](http://www.cs.uic.edu/~liub/FBS/opinion-lexicon-English.rar). This is based on nearly 
68,000 words from the English Language and words categorised to be positive or negative.

Using a simple matching algorithm a _sentiment score_ can be computed. This boolean match of each
word is done for each Tweet, the total positive sentiment score minus the total negative sentiment 
score.

<!-- jsHeader -->
<script type="text/javascript">
 
// jsData 
function gvisDataColumnChartIDeb47e2d9e4 () {
var data = new google.visualization.DataTable();
var datajson =
[
 [
 "-3",
4 
],
[
 "-2",
5 
],
[
 "-1",
80 
],
[
 "0",
554 
],
[
 "1",
349 
],
[
 "2",
111 
],
[
 "3",
23 
] 
];
data.addColumn('string','Var1');
data.addColumn('number','Freq');
data.addRows(datajson);
return(data);
}
 
// jsDrawChart
function drawChartColumnChartIDeb47e2d9e4() {
var data = gvisDataColumnChartIDeb47e2d9e4();
var options = {};
options["allowHtml"] = true;
options["legend"] = "none";
options["vAxes"] = [{title:'Freq'}];
options["hAxes"] = [{title:'Sentiment Score'}];
options["width"] =    750;
options["height"] =    400;


    var chart = new google.visualization.ColumnChart(
    document.getElementById('ColumnChartIDeb47e2d9e4')
    );
    chart.draw(data,options);
    

}
  
 
// jsDisplayChart
(function() {
var pkgs = window.__gvisPackages = window.__gvisPackages || [];
var callbacks = window.__gvisCallbacks = window.__gvisCallbacks || [];
var chartid = "corechart";
  
// Manually see if chartid is in pkgs (not all browsers support Array.indexOf)
var i, newPackage = true;
for (i = 0; newPackage && i < pkgs.length; i++) {
if (pkgs[i] === chartid)
newPackage = false;
}
if (newPackage)
  pkgs.push(chartid);
  
// Add the drawChart function to the global list of callbacks
callbacks.push(drawChartColumnChartIDeb47e2d9e4);
})();
function displayChartColumnChartIDeb47e2d9e4() {
  var pkgs = window.__gvisPackages = window.__gvisPackages || [];
  var callbacks = window.__gvisCallbacks = window.__gvisCallbacks || [];
  window.clearTimeout(window.__gvisLoad);
  // The timeout is set to 100 because otherwise the container div we are
  // targeting might not be part of the document yet
  window.__gvisLoad = setTimeout(function() {
  var pkgCount = pkgs.length;
  google.load("visualization", "1", { packages:pkgs, callback: function() {
  if (pkgCount != pkgs.length) {
  // Race condition where another setTimeout call snuck in after us; if
  // that call added a package, we must not shift its callback
  return;
}
while (callbacks.length > 0)
callbacks.shift()();
} });
}, 100);
}
 
// jsFooter
</script>
 
<!-- jsChart -->  
<script type="text/javascript" src="https://www.google.com/jsapi?callback=displayChartColumnChartIDeb47e2d9e4"></script>
 
<!-- divChart -->
  
<div id="ColumnChartIDeb47e2d9e4" 
  style="width: 750; height: 400;">
</div>

{% highlight r %}
# Mean of #peakdistrict score
mean(PDNP_Tweets_Result$score, trim = 0, na.rm = TRUE)
[1] 0.472

## Standard Deviation of #peakdistrict score
sd(PDNP_Tweets_Result$score, na.rm = FALSE)
[1] 0.8839029
{% endhighlight %}

From the observations, it is clear the basic sentiment analysis doesn't provide us detailed information. I have extracted two positive and two negative tweets below. Although the mean is positive, the majority of tweets are actually neutral.  

{% highlight r %}
# Select tweets with -3 scores
Negative_Score <- PDNP_Tweets_Result$text[ which(PDNP_Tweets_Result$score==-3)]
Negative_Score
[1] 'planning a walk for tomorrow but having a mile limit before the nerve pain really kicks in is rather restrictive…'
[2] 'appalled by persecution of any species very difficult to make any detail at all compare and disc…'

# Select tweets with 3 scores
Positive_Score <- PDNP_Tweets_Result$text[ which(PDNP_Tweets_Result$score==3)]
Positive_Score
[1] 'do love the peak district so many places to walk fantastic scenery and great eating out'
[2] 'combine a stay in the with learning a new skill inspiring scenery and inspiring courses peak'
{% endhighlight %}

### _Naive Bayes_

One of the key problems of analytics is to classify entities or events based on a knowledge of their 
attributes. Rather than a simple matching of opinion lexicon, the [_Naive Bayes_](https://en.wikipedia.org/wiki/Bayes%27_theorem) method helps decide on and classify a series of emotionions present in each Tweet.

This uses the ['Rstem'](https://cran.r-project.org/web/packages/Rstem/) and ['Sentiment'](https://cran.r-project.org/web/packages/sentiment/index.html) package. The 'Sentiment' package was built to use a _trained_ dataset of emotion words (approximatley 1,500). Results can then be generated belonging to one of six emotions:
**anger**, **disgust**, **fear**, **joy**, **sadness** and **surprise**. Not all tweets will contain
data to fit these categories and get disregarded from the analysis.

Currently getting the Error... {% highlight r %}in simple_triplet_matrix(i = i, j = j, v = as.numeric(v), nrow = length(allTerms), :{% endhighlight %} waiting for a resolution.
