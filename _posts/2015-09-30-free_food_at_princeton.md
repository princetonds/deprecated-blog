---
layout: post
title: Free Food at Princeton 
---
<h6>*Author: Felix Xiao*

##*Predicting Free Food at Princeton*

Princeton has a [Free Food Listserv](http://www.universitypressclub.com/archive/2011/04/usg-knows-what-we-want/). Here’s a brunch of emails sent on it a while ago:

```{r}
> data = read.csv(‘freefood.csv’)> head(data)        date hour min                                                             subj1 2013-11-20   16  55                                [FreeFood] Olives at campus club!2 2013-11-20   19  15                     [FreeFood] Free Pizza in Lewis Library Lobby3 2013-11-21   16   4                  [FreeFood] FREE PiZZA in Frist by the TV lounge4 2013-11-21   16  15 [FreeFood] Wine, Brownies, Miscellaneous Snacks in Schultz Lobby5 2013-11-22   12   9                               [FreeFood] Sandwiches in Frist 2476 2013-11-23   17  56                           [FreeFood] CHINESE FOOD IN CAMPUS CLUB```
This data set lists the dates, times, and subject lines of all emails sent on the Free Food Listserv since **November 20, 2013**. It can be downloaded [here] (https://github.com/princeton-data-science/FreeFood).
In this blog post, I’ll be analyzing these emails with the statistical programming language, [R] (https://www.r-project.org/). I’ve embedded my code in the analysis to show exactly how I’m doing that. Feel free to download my code and run it yourself.

Data science is about coming up with interesting, well-defined questions and manipulating data to find rigorous answers. For instance, **how many emails are sent out on the Free Food Listserv per day, on average?** The answer depends a lot on whether we include winter break, summer vacation, etc. Virtually no emails are sent out during breaks, so we can sensibly restrict our data set to the 2014-2015 academic year.

```{r}> school.days = seq(as.Date('2014-09-10'), as.Date('2014-10-24'), by = 1)> school.days = c(school.days, seq(as.Date('2014-11-03'), as.Date('2014-11-25'), by = 1))> school.days = c(school.days, seq(as.Date('2014-12-01'), as.Date('2014-12-12'), by = 1))> school.days = c(school.days, seq(as.Date('2015-02-02'), as.Date('2015-03-14'), by = 1))> school.days = c(school.days, seq(as.Date('2015-03-23'), as.Date('2015-05-03'), by = 1))>> data = data[data$date %in% school.days,]```
Now we can answer our question by simply dividing the number of emails by the number of school days.```{r}> nrow(data) / length(school.days)[1] 2.441718
```
That’s not too bad. On the face of it, I could grab several snacks or even full meals per day by just checking my email religiously and running very fast. We should be cautious drawing this kind of conclusion however, since we only know the average. For all we know, half the days may see zero emails and the other half five. It’s important to look at the **distribution of number of emails per day**. For that, we need to coax our data set from a list of dates into a table denoting, for each school day, how many emails were sent out. Then we plot a histogram.


```{r}> # Create a table of 0s with length the number of school days> count = rep(0, length(school.days))> names(count) = school.days>> # On days emails were sent, fill in the rows of the table with the numbers of emails sent> count[names(table(data$date))] = table(data$date)>> # Verify the mean is the same> mean(count)[1] 2.441718> hist(count)```

<div style="middle;width:70%;height:70%;margin-right:auto; margin-left:auto;">
	<center><img style="middle;width:100%;height:100%" src="{{ site.baseurl }}free_food_hist.jpg"></center>
</div>

Roughly 2.4 emails were sent out on average, but by no means consistently. The histogram tells us that no emails were sent out about a third of the days. Someone looking to subsist on the Free Food listserv must either be resigned to an empty stomach on those days or stash food when 4 or more emails are sent out (approx. 15% of the time from the histogram). Incidentally, the distribution looks sort of [geometric] (https://en.wikipedia.org/wiki/Geometric_distribution). On the tail end, there was one day that saw 11 emails on the listserv.

```{r}
> feast.day = names(which.max(count))> data[data$date == feast.day,]659 2014-12-10   14   4                                  [FreeFood] Food Panel and Senegalese Food660 2014-12-10   15  11                            [FreeFood] 2 boxes of pizza + 1 bottle of pepsi661 2014-12-10   15  13                                                                 [FreeFood]662 2014-12-10   15  38                   [FreeFood] Boom chicka pop, Chips & salsa in pace center663 2014-12-10   16  45                                 [FreeFood] Free Mehek!! Robertson basement664 2014-12-10   17  19                         [FreeFood] Senegalese food at Carl a fields center665 2014-12-10   17  20                                                 [FreeFood] Senegalese food666 2014-12-10   17  42                         [FreeFood] Reception leftovers in art museum lobby667 2014-12-10   18  54                                   [FreeFood] COFFEE, CANDY, COOKIES, CHIPS668 2014-12-10   19  41                                  [FreeFood] Chinese in Whitman Common Room669 2014-12-10   19  51              [FreeFood] Cake on top of recycle bin on third floor of clapp670 2014-12-10   20  35 [FreeFood] Egg Nog, Cookies, Chips, Apple Cider, Pumpkin Pie, Candy Canes!```

That glorious day was Dec 10, 2014, a Wednesday. Perhaps more emails are sent out on certain days of the week? Let’s **plot the mean number of emails for each day of the week**.

```{r}
> # split data frame to list of data frames by day of week> day.of.week = split(data, weekdays(data$date))>> # order list Sunday, Monday, ... Saturday> dow = c('Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday')> day.of.week = day.of.week[dow]>> # create table for number of emails sent and total number of days> emails = sapply(day.of.week, nrow)> days   = table(weekdays(school.days))> days   = days[dow]>> barplot(emails / days, main = 'Avg # emails each day of week')```
<div style="middle;width:70%;height:70%;margin-right:auto; margin-left:auto;">
	<center><img style="middle;width:70%;height:70%" src="{{ site.baseurl }}avg_emails_free_food.jpg"></center>
</div>
To see whether these differences in average emails are significant or are just noise, we need to draw some confidence intervals. We’ll start by making some assumptions:1.	For each day of the week D, the numbers of emails $X^{(D)}_1, X^{(D)}_2,....,X^{(D)}_{nD}$ sent on that day all follow the same probability distribution with mean μD and standard deviation σD.2.	The number of emails sent on any day does not depend on the numbers of emails sent on any past daysTaken together, these are the *independent(2) and identically distributed (1)* assumptions on data that ~ 90% of statistical techniques rely on. This page gives another succinct explanation of [independent, identically distributed] (http://math.stackexchange.com/questions/466927/independent-identically-distributed-iid-random-variables) random variables.

The central limit theorem says that, as nD increases the quantity $$\frac{1}{nD} \sum_{i=1}^{nD} X^{(D)}_i$$  (which is the average of the data values) gets closer and closer to a [Gaussian distribution] (http://mathworld.wolfram.com/NormalDistribution.html) with mean μD and standard deviation $\frac{ \sigma D}{ \sqrt{nD}}$.

This just means that the average number of emails sent on Fridays differs from the “real” average by something around $\frac{ \sigma D}{ \sqrt{nD}}$. If nD increases and we have more Fridays from which to collect listserv data, then the difference will shrink. This is why more data leads to better estimates. For an (approximately) Gaussian distribution, {% latex %}\frac{1}{nD} \sum_{i=1}^{nD} X^{(D)}_i{% endlatex %} will be within $\frac{ \sigma D}{ \sqrt{nD}}$ units about 68% of the time. So if we draw on our graph brackets centered on 

$$\frac{1}{nD} \sum_{i=1}^{nD} X^{(D)}_i$$ with radius $\frac{ \sigma D}{ \sqrt{nD}}$ we’ve just constructed a ~68% confidence interval.Aside: we actually don’t know what $σ_D$ is but we can approximate it with  
 {% latex %} \sqrt{\frac{1}{n_D}\sum_{i=1}^{n_D}(x_i^{(D)}-average(x^{(D)}))^2}{% endlatex %},
and the resulting estimate of $\frac{ \sigma D}{ \sqrt{nD}}$ is called the *standard error of the mean*. Why this approximation? 

Because $\sigma D^2$ is defined as 
$$ E[(X^{(D)}-µ_D)^2]$$
 by the law of large numbers,
 $$ \frac{1}{n_D}\sum_{i=1}^{n_D}(x_i^{(D)}-average(x^{(D)}))^2$$
converges to $\sigma D^2$. Now we’re ready to add error bars to the plot.
```{r}> # create a function to compute standard error for a> # set of emails sent on a particular day> compute.se = function(df)> {>   dow = unique(weekdays(df$date))>   stopifnot(length(dow) == 1)>   >   days = school.days[weekdays(school.days) == dow]>   n = length(days)>   >   count = rep(0, length(days))>   names(count) = days>   count[names(table(df$date))] = table(df$date)>   >   return( sd(count) / sqrt(n) )  > }>> se = sapply(day.of.week, compute.se)>> # barplot( ) returns horizontal position of the days> x  = barplot(emails / days, main = 'Avg # emails each day of week', ylim = c(0, 5))> arrows(x, emails / days + se, x, emails / days - se, angle = 90, code = 3, length = 0.1)```
<div style="middle;width:70%;height:70%;margin-right:auto; margin-left:auto;">
	<center><img style="middle;width:70%;height:70%" src="{{ site.baseurl }}avg_emails.jpg"></center>
</div>

<p>Since Friday’s 68% confidence interval doesn’t overlap with that of any other day of the week, we can safely say emails are sent out more frequently on Fridays. On the other hand the Sunday, Wednesday, Thursday, Saturday intervals overlap by a lot, so the corresponding population means are likely not that different.</p><p>What about time of day?</p>```{r}> hist(data$hour, breaks = seq(-0.5, 23.5, 1), xlab = 'Hour of day', freq = F,>      main = 'Free food emails by hour of day', ylab = 'Proportion of emails', xaxt = 'n')> axis(side = 1, at = seq(-0.5, 23.5, length.out = 9),>      labels = c('12am', '3am', '6am', '9am', '12pm', '3pm', '6pm', '9pm', '12am'))```
<div style="middle;width:70%;height:70%;margin-right:auto; margin-left:auto;">
	<center><img style="middle;width:70%;height:70%" src="{{ site.baseurl }}emails_perhour.jpg"></center>
</div>
The distribution is somewhat bimodal; there’s a small cluster scattered around noon and a larger cluster scattered around 6pm. Not many to expect from 11pm to 10am, which makes sense. I was curious about that one email sent out at 3 in the morning, so I checked:```{r}
> data[data$hour == 3,]          date hour min                         subj752 2015-02-15    3  17 [FreeFood] wallet at terrace```
It turns out a small fraction of the emails sent to the listserv are unrelated to food. Not enough to justify the trouble of filtering out all the non-food related ones, but it’s just something to keep in mind.Now let’s see what people are putting on the subject lines. I’m going to do the simple task of collecting all words from all email subject lines and looking at which words ones come up most often (I also did some preprocessing so words like “Pizza” and “PIZZA” and “PiZZa!” are treated as the same). The function **subject.line.words** does this```{r} 
subject.line.words = function(data){  words = unlist(lapply(data$subj, function(subj) strsplit(subj, ' ')))  # split subject line into individual words  words = words[words != '[FreeFood]']  words = tolower(words)  # make all words lowercase  words = sapply(words, function(word) gsub("[[:punct:]]", "", word))  # delete any punctuation marks  words.freq = table(words)  return(sort(words.freq, decreasing = T))}```
Let’s look at all the emails first. The 30 most common subject line words are:```{r}
> subject.line.words(data)[1:30]words        in        and                  food      frist      pizza       free    cookies sandwiches         of        156        103         64         64         59         58         54         44         44         41        the         at      salad       room     center      chips      dodge       from     murray       club         31         26         26         25         23         22         20         20         20         18     campus     friend     lounge     olives    chicken      floor    outside   basement    chinese     common         17         17         17         17         14         14         14         13         13         13```
The subject lines are quite informative about (1) location of free food and (2) the type of food. Looking at the table above, we see that Frist is a popular venue (59 emails), along with Murray-Dodge (20), Friend Center (17), and Campus Club (17). We also note that Pizza (58) is the most-supplied food item, followed by cookies (44) and sandwiches (44).We can take this in a number of directions. We can find out which foods are closely associated with what locations. We can see if the time of day the email was sent has any effect on subject line content. A fast way of answering the latter question is to simply filter for emails sent during a certain period in the day and call subject.line.words. We do that with emails sent before 3pm:```{r}
> subject.line.words(data[data$hour < 15,])[1:30]words        in        and            sandwiches       food       free      frist         at      salad    cookies         49         38         27         24         18         17         15         14         14         13         of        the     center     friend      pizza     bagels     lounge     mcgraw    rainbow     campus         12         12         10         10         10          8          8          8          8          6    chicken       club     coffee  chocolate       from       lgbt       room    tonight      wraps      bread          6          6          6          5          5          5          5          5          5          4```
And emails sent after 3pm:

```{r}
> subject.line.words(data[data$hour >= 15,])[1:30]words        in        and      pizza       food      frist                  free    cookies         of       room        107         65         48         46         44         37         37         31         29         20 sandwiches      chips        the      dodge     murray       from     olives      floor     center         at         20         19         19         17         17         15         15         14         13         12   basement       club     common      salad     campus    chinese    outside     indian     lounge     cheese         12         12         12         12         11         11         11         10          9          8```
<p>This quickly shows that there is a relationship between time of day and subject line content. Namely, that sandwiches are more prevalent during lunch hours and pizza during dinner hours. We can also see that Friend Center is pretty common location before 3pm, but hardly used at all after 3pm.</p>Let’s do something more sophisticated and frame the same question as a regression problem. We are looking for a math function whose input will be the words in the subject line of an email and whose output will be the best guess for the hour of day the email was sent. For reasons of model flexibility, high dimensionality, and interpretability, we’re going to model the regression function as a classic regression tree. If you haven’t heard of a regression tree before, please read [this page] (http://www.r2d3.us/visual-intro-to-machine-learning-part-1/) before reading the next section.To train the regression tree, the matrix of input variables **X** must be in this form: each row $i$ is an individual email and each column $j$ is a word. The matrix entry $x_{ij}$ will be 1 if word $j$ is in the subject line of email $i$, and 0 otherwise. **X** will have number of columns equal to how many unique words are in all the email subjects. In natural language processing, **X** is called a *term document matrix*.

```{r}
> library(tm)> subject.lines = gsub('[[:punct:]]', ' ', data$subj)> subject.lines = gsub('FreeFood', '', subject.lines)> corpus = VCorpus(VectorSource(subject.lines))> tdm = TermDocumentMatrix(corpus)> tdm = as.data.frame(t(as.matrix(tdm)))```In our case, as typically, the term document matrix is sparse (most entries are 0) and high-dimensional (more variables than observations, or more columns than rows). Now we train our regression tree with the package *rpart*.```{r}> library(rpart)> tdm$y = data$hour + data$min / 60> tree = rpart(y ~ ., data = tdm)> par(xpd = NA)> plot(tree)> text(tree)```
<div style="middle;width:70%;height:70%;margin-right:auto; margin-left:auto;">
	<center><img style="middle;width:80%;height:80%" src="{{ site.baseurl }}freefood_tree.jpg"></center>
</div>

In the regression tree above, each fork has a conditional. For instance, bagels >= 0.5 means that the word “bagels” was in the email subject line. If the conditional is true, then you move to the left child node; if false, move to the right one. This step repeats until you reach a terminal node, upon which the predicted time in hours after midnight is given.“Bagels” turns out to be the single best predictor of time of all the words, predicting 11:24am if present. If “bagels” is not in the subject line, then “sandwiches” is the next best predictor, predicting 2:26pm. If neither “sandwiches” nor “bagels” are in the subject line, then emails having “friend” in the subject line tend to be sent in the early afternoon (1:56pm). Emails with subjects lacking “bagels”, “sandwiches”, and “friend” are all sent later in the afternoon and in the evening.Our last item to investigate is assocations between words in the free food email subjects lines. Given any two words, say “Sushi” and “Frist” we want to know how often they tend to appear together in an email subject. We want to create an association matrix **A**. **A** will be symmetric and each row and column will be assigned a word. For each row $i$ and column $j$ in the matrix, the entry
$$a_{ij}=\frac{\text{# emails with both word i and j}} {\text{# emails with either word i or j}}$$

We restrict our scope of words to those that appeared in at least 6 emails. After we have our assocation matrix **A** (which maps 63 words) we visualize it as a network graph. Nodes are words and edges between words indicate an **A** entry of at least 0.15 (thicker edges indicate greater assocation).

```{r}
> m = as.matrix(tdm[,-ncol(tdm)]) > 0> colsums = apply(m, 2, sum)> m = m[, colsums >= 6]>> X = t(m) %*% m> Y = t(!m) %*% m> A = X / (X + Y + t(Y))>> library(qgraph)> qgraph(A, minimum = 0.15, border.color = 'Gray', labels = colnames(S), label.scale = F, label.cex = 0.8)```
<div style="middle;width:70%;height:70%;margin-right:auto; margin-left:auto;">
	<center><img style="middle;width:80%;height:80%" src="{{ site.baseurl }}freefood_graph.jpg"></center>
</div>


Some interesting ones I found by eyeballing this graph:
*	panera – lewis
* 	rice – chicken*	rainbow – lounge (where is that?)*	indian – food – murray – dodge*	tea – cupcakes


----------------------------<p>Acknowledgements: Evan Chow ‘16, for providing both the data set and the initial idea for its analysis. This blog post would not exist without your abiding enthusiasm for free food.</p>