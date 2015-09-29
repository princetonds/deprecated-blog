---
layout: post
title: Free Food at Princeton 
---
<h6>*Author: Felix Xiao*

##*Predicting Free Food at Princeton*

Princeton has a [Free Food Listserv](http://www.universitypressclub.com/archive/2011/04/usg-knows-what-we-want/). Here’s a brunch of emails sent on it a while ago:

```{r}
> data = read.csv(‘freefood.csv’)
This data set lists the dates, times, and subject lines of all emails sent on the Free Food Listserv since **November 20, 2013**. It can be downloaded [here] (https://github.com/princeton-data-science).


Data science is about coming up with interesting, well-defined questions and manipulating data to find rigorous answers. For instance, **how many emails are sent out on the Free Food Listserv per day, on average?** The answer depends a lot on whether we include winter break, summer vacation, etc. Virtually no emails are sent out during breaks, so we can sensibly restrict our data set to the 2014-2015 academic year.

```{r}
Now we can answer our question by simply dividing the number of emails by the number of school days.
```
That’s not too bad. On the face of it, I could grab several snacks or even full meals per day by just checking my email religiously and running very fast. We should be cautious drawing this kind of conclusion however, since we only know the average. For all we know, half the days may see zero emails and the other half five. It’s important to look at the **distribution of number of emails per day**. For that, we need to coax our data set from a list of dates into a table denoting, for each school day, how many emails were sent out. Then we plot a histogram.


```{r}

<div style="middle;width:70%;height:70%;margin-right:auto; margin-left:auto;">
	<center><img style="middle;width:50%;height:50%" src="{{ site.baseurl }}free_food_hist.jpg"></center>
</div>

Roughly 2.4 emails were sent out on average, but by no means consistently. The histogram tells us that no emails were sent out about a third of the days. Someone looking to subsist on the Free Food listserv must either be resigned to an empty stomach on those days or stash food when 4 or more emails are sent out (approx. 15% of the time from the histogram). Incidentally, the distribution looks sort of [geometric] (https://en.wikipedia.org/wiki/Geometric_distribution). On the tail end, there was one day that saw 11 emails on the listserv.

```{r}
> feast.day = names(which.max(count))

That glorious day was Dec 10, 2014, a Wednesday. Perhaps more emails are sent out on certain days of the week? Let’s **plot the mean number of emails for each day of the week**.

```{r}
> # split data frame to list of data frames by day of week
<div style="middle;width:70%;height:70%;margin-right:auto; margin-left:auto;">
	<center><img style="middle;width:50%;height:50%" src="{{ site.baseurl }}avg_emails_free_food.png"></center>
</div>



The central limit theorem says that, as nD increases the quantity $$\frac{1}{nD} \sum_{i=1}^{nD} X^{(D)}_i$$  (which is the average of the data values) gets closer and closer to a [Gaussian distribution] (http://mathworld.wolfram.com/NormalDistribution.html) with mean μD and standard deviation $\frac{ \sigma D}{ \sqrt{nD}}$.

This just means that the average number of emails sent on Fridays differs from the “real” average by something around $\frac{ \sigma D}{ \sqrt{nD}}$. If nD increases and we have more Fridays from which to collect listserv data, then the difference will shrink. This is why more data leads to better estimates. For an (approximately) Gaussian distribution, {% latex %}\frac{1}{nD} \sum_{i=1}^{nD} X^{(D)}_i{% endlatex %} will be within $\frac{ \sigma D}{ \sqrt{nD}}$ units about 68% of the time. So if we draw on our graph brackets centered on 

$$\frac{1}{nD} \sum_{i=1}^{nD} X^{(D)}_i$$ with radius $\frac{ \sigma D}{ \sqrt{nD}}$ we’ve just constructed a ~68% confidence interval.
 {% latex %} \sqrt{\frac{1}{n_D}\sum_{i=1}^{n_D}(x_i^{(D)}-average(x^{(D)}))^2}{% endlatex %},
and the resulting estimate of $\frac{ \sigma D}{ \sqrt{nD}}$ is called the *standard error of the mean*. Why this approximation? 

Because $\sigma D^2$ is defined as 
$$ E[(X^{(D)}-µ_D)^2]$$
 by the law of large numbers,
 $$ \frac{1}{n_D}\sum_{i=1}^{n_D}(x_i^{(D)}-average(x^{(D)}))^2$$
converges to $\sigma D^2$. Now we’re ready to add error bars to the plot.

<div style="middle;width:70%;height:70%;margin-right:auto; margin-left:auto;">
	<center><img style="middle;width:50%;height:50%" src="{{ site.baseurl }}avg_emails.png"></center>
</div>

<p>Since Friday’s 68% confidence interval doesn’t overlap with that of any other day of the week, we can safely say emails are sent out more frequently on Fridays. On the other hand the Sunday, Wednesday, Thursday, Saturday intervals overlap by a lot, so the corresponding population means are likely not that different.</p>
<div style="middle;width:70%;height:70%;margin-right:auto; margin-left:auto;">
	<center><img style="middle;width:50%;height:50%" src="{{ site.baseurl }}emails_perhour.png"></center>
</div>

> data[data$hour == 3,]
It turns out a small fraction of the emails sent to the listserv are unrelated to food. Not enough to justify the trouble of filtering out all the non-food related ones, but it’s just something to keep in mind.
subject.line.words = function(data)

> subject.line.words(data)[1:30]
The subject lines are quite informative about (1) location of free food and (2) the type of food. Looking at the table above, we see that Frist is a popular venue (59 emails), along with Murray-Dodge (20), Friend Center (17), and Campus Club (17). We also note that Pizza (58) is the most-supplied food item, followed by cookies (44) and sandwiches (44).

And emails sent after 3pm:

```{r}
> subject.line.words(data[data$hour >= 15,])[1:30]
<p>This quickly shows that there is a relationship between time of day and subject line content. Namely, that sandwiches are more prevalent during lunch hours and pizza during dinner hours. We can also see that Friend Center is pretty common location before 3pm, but hardly used at all after 3pm.</p>

```{r}

<div style="middle;width:70%;height:70%;margin-right:auto; margin-left:auto;">
	<center><img style="middle;width:50%;height:50%" src="{{ site.baseurl }}"freefood_tree.png"></center>
</div>

In the regression tree above, each fork has a conditional. For instance, bagels >= 0.5 means that the word “bagels” was in the email subject line. If the conditional is true, then you move to the left child node; if false, move to the right one. This step repeats until you reach a terminal node, upon which the predicted time in hours after midnight is given.
$$a_{ij}=\frac{\text{# emails with both word i and j}} {\text{# emails with either word i or j}}$$




> m = as.matrix(tdm[,-ncol(tdm)]) > 0
<div style="middle;width:70%;height:70%;margin-right:auto; margin-left:auto;">
	<center><img style="middle;width:50%;height:50%" src="{{ site.baseurl }}"freefood_graph.png"></center>
</div>


Some interesting ones I found by eyeballing this graph:

* 	rice – chicken


