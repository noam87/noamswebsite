+++
date = "2014-01-23"
title = "Sorting Posts by User Engagement Level"
draft = false
wikis = ["datascience", "math"]
tags = ["featured"]
+++

At Functional Imperative we're building the new *CanLII Connects*
website (a social portal for Canada's largest database of legal cases), and
this week I was given the task of figuring out a sensible way of sorting posts.

Figuring out how to sort user-generated content is a common problem that many
social websites face.

Here's Reddit's scoring equation for 'Best' <a href="http://www.evanmiller.org/how-not-to-sort-by-average-rating.html">(source and explanation)</a>:

![Reddit's 'best' scoring equation](/img/reddit\_best.png)

Not all scoring equations are that hairy, <a href="http://moz.com/blog/reddit-stumbleupon-delicious-and-hacker-news-algorithms-exposed">here are a few more</a>.

Interestingly enough, Reddit's 'Hot' scoring function (explained in link above):

![Reddit's "hot" algorithm](/img/reddit\_hot\_algo.png)

is <a href="http://technotes.iangreenleaf.com/posts/2013-12-09-reddits-empire-is-built-on-a-flawed-algorithm.html">quite flawed</a>.

> <strong>Sidenote</strong>: One observation not mentioned in that first
> article is that, while all other equations use some form of `time_now -
> time_posted` to calculate how old a post is, the clever guys at Reddit use
> `time_posted - some_old_date`.
>
> The advantage of this is that the post's score need only be calculated once,
> whereas the value of scores calculated with `time_now` will change on every
> request.

Anyway, while all those scoring functions work pretty well, they didn't quite
fit the requirements for *CanLII Connects*.

In this post, I'll walk through the decision process of creating a scoring
function. Hopefully this will be useful if you encounter a similar feature to
implement.

## Requirements:

*CanLII Connects* links to a database of legal cases, and users can post
opinions on those cases:

<ol>
  <li>A user can post.</li>
  <li>A user can upvote a post.</li>
  <li>A user can comment on a post.</li>
  <li>A user can comment on a comment on a post.</li>
</ol>

So what's a sensible way of sorting posts?

Right away, we're dealing with a different problem than Reddit or HN: while it
makes sense to slowly degrade the score of a post on those sites over time, the
same does not make sense for CanLII. Old cases might be cited at any time, no
matter how old they are, so what matters is not how old a discussion is, but
rather how actively engaged users are within a given discussion.

## Initial Score

Ok, so first we have to give each post an initial score. I like Reddit's
approach of taking the base-10 log of its upvotes. This makes sense because,
the more popular a post already is, the more likely people are to see it, and
therefore upvote it, which gives it an unfair advantage.

In our case, we're not only trying to measure how much people "like" a post,
but rather how engaged they are with it.  It makes sense that, while an upvote
is a fine indicator of "engagedness", a user actually bothering to comment on a
post is even more of an indicator. I'll count that as equivalent to two
upvotes, and a user commenting on a comment will count as three upvotes (the 2
is so we don't take the log of 1 or 0):

$$log\_{10}(2 + u + 2c + 3cc)$$

## Frequency

Next, we need the post's position to degrade as it becomes less active. It
makes sense to divide the intial score by some factor of time:

$$\cfrac{log\_{10} (2+u+2c+3cc)}{\bar{t}}$$

Now we need a reasonable value for $\bar{t}$.  A good start is the average
time, in seconds, between the three most recent user interactions with  a post.

We define a user interaction to be: a user creates a post, a user comments on a
post, or a user upvotes a post.

Also, we want the most recent interactions to weigh more than older
interactions.  So let's say each `t` weighs twice as much as the previous:

$$\bar{t} = \cfrac{\sum\_{i=1}^3 \left(\frac{1}{2}\right)^{i-1} * (t\_i - t\_{i-1})}{\sum\_{i=1}^3  \left(\frac{1}{2}\right)^{i-1}}$$

Where

$t\_0 $ = <a href="http://en.wikipedia.org/wiki/Unix\_time">UNIX timestamp</a>,
at now, in seconds.

$t\_n  $ = <a href="http://en.wikipedia.org/wiki/Unix\_time">UNIX timestamp</a>
of n<sup>th</sup> interaction.

## One Final Detail

There is one last property we want this function to have, which is the
following: if interactions are very frequent right now (within a timeframe of,
say, 10 days), then clearly the post is "hot", and its score should be boosted.
But as time passes, it really doesn't matter as much how much distance there is
between interactions. If a post has already gone a full year without anyone
commenting on it, does it really make that much difference if it goes another
month without a comment?

To accomplish the first property, all we do is divide $\bar{t} $ by the number
of seconds in 10 days: `60*60*24*10`.

To accomplish the second property, what we are looking for is some sort of
always-increasing, concave function (positive derivative, negative second
derivative).  The first thing that comes to mind is the square-root function,
which is good enough.

## Result

And thus we have our final scoring function:

$$\cfrac{log\_{10} (2 + u + 2c + 3cc)}{\sqrt{\bar{t}/60\*60\*24\*10}}$$

$$\bar{t} = \cfrac{\sum\_{i=1}^3 \left(\frac{1}{2}\right)^{i-1} \* (t\_i - t\_{i-1})}{\sum\_{i=1}^3  \left(\frac{1}{2}\right)^{i-1}}$$

If we plot this equation for `x = number of points` and `y = time`, we can see
the shape of this function and check for different values if they make sense:

![Scoring function 3D plot](/img/scoring_function_shape_2.jpg)

As expected, there is a steep 10-day "boost" period, followed by an
increasingly slower decline in the value as more and more time passes.

> The function is also heavily biased toward very new posts, which will always
> come out on top, giving them a chance. This might be a bad idea if posting
> becomes frequent, but user interaction is low (many summaries a day, few
> votes or comments), and might have to be changed.
>
> There are many ways to tweak this equation (changing the boost period, for
> example) to make it more or less biased towards either time or user
> interaction.

---

## Bonus Round: Implementing in ElasticSearch

Implementing a custom scoring function in Elasticsearch, though easy once it's
all set up, was rather frustrating because of the poor documentation.

For our implementation, we're using the `tire` gem (a wrapper around the
Elasticsearch API). This is where we call the custom scoring script:

```
query do
  #custom_score script: "parseInt(doc['upvote_count'].value)", lang: "javascript" do
  custom_score script: script, lang: 'javascript' do
    string query.join(" OR ")
  end
end
```

Where `script` is simply a variable holding the contents of a javascript file
as a string. Note the option `lang: 'javascript'`. This lets us use javascript
as our language of choice, as opposed to
<ahref="http://mvel.codehaus.org/">mvel</a>,
the most poorly documented scripting
language on the face of the earth. To enable this option, we'll also require
the
<a href="https://github.com/elasticsearch/elasticsearch-lang-javascript">elasticsearch-lang-javascript</a> plugin.

Here is our script:

> <strong>Sidenote:</strong> Notice the logger function. This enables us to implement a sort of "console.log" which we can read using the following shell command `tail -f /var/log/elasticsearch/elasticsearch.log`.

```
// Logger function:
var logger = org.elasticsearch.common.logging.Loggers.getLogger("rails_logger");
// Example usage:
logger.info("========= NEW CALC ===========");

var points_log = parseFloat(doc.points_log.value);
var now = Math.round(new Date().getTime() / 1000);

/**
* NOTE: doc.ts.values is not actually an array,
* here I create an array out of it:
**/
var ts = [];
for (var i = 0; i < doc.ts.values.length; i++) ts[i] = doc.ts.values[i];
ts.push(now);
// Newest first
ts.reverse();

/**
* Boost period.
**/
var ten_days = 60*60*24*10;

/**
* The scoring function
**/
function score() {
  /**
  * Weighed average numerator
  **/
  var times_num = (function() {
    var val = 0;
    for (var i = 1; i < ts.length; i++) {
      var exp = i - 1;
      val += Math.pow(0.5, exp) *
             (parseFloat(ts[i]) -
             parseFloat(ts[i - 1]));
    }
    return val;
  })();

  /**
  * Weighed average denominator
  **/
  var times_denom = (function() {
    var val = 0;
    for (var i = 1; i < ts.length; i++) {
      var exp = i - 1;
      val += Math.pow(0.5, exp);
    }
    return val;
  })();

  var t_ave = (times_num/times_denom);

  return points_log/Math.sqrt(t_ave/ten_days);
};

score();
```
