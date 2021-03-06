---
layout: post
title:  "What we can learn from 20 years of ATP match data"
date:   2017-06-12 13:48:34 +1000
categories: assessment
---

![Code]({{ site.url }}/assets/codeball.png)

### A lucky find

I recently stumbled upon a trove of tennis match statistics, and being a lover of the game I knew that I had to analyse it. The last 15 years have been a golden age of tennis, where a handful of players have shown unprecedented dominance. Given the dataset covered every ATP match over this period, I knew there would be some interesting trends to uncover.

It was quite overwhelming trying to decide where to get started, as there was data for around ~40 different fields. I knew that I wanted to explore the impact of Big 4, so I chose to look at how the game has changed since the year 2000, focusing on age, rank points, and aces. I also wanted to delve a bit deeper into the data and compare the 4 grand slams.

If you're interested in having a look at the data yourself, it's available [here](https://github.com/JeffSackmann/tennis_atp). Credit to [JeffSackman](github.com/JeffSackmann) for doing a great job compiling the statistics.


### Is tennis getting older?

In case you're unfamiliar with the term 'the Big Four', it refers to Roger Federer, Rafael Nadal, Novak Djokovic, and Andy Murray, and their complete dominance in rankings and tournament victories since 2004. [Wikipedia](https://en.wikipedia.org/wiki/Big_Four_(tennis)) has a very extensive article on the subject if you want to dive deeper.

Despite their seemingly inhumane achievements, they are still bound by the laws of nature, specifically that their age increases linearly with time. To see this manifested in the data, I plotted the overall winners average age for every Masters and Grand Slam level tournament since 2000.

![Avg. Age of Tourney Winner]({{ site.url }}/assets/age_graph.png)

Unsurprisingly, we see a clear trend of age increasing. One would assume that this trend won't continue for much longer, although as there are no clear successors (except Nick Kyrgios), who knows what will happen. Considering that this years Australian and French Open were won by Federer (age 35), and Nadal (age 31) respectively, they all seem to showing no signs of slowing down.

### Few rule the many

Another way to see just how dominant these players have been is to examine the distribution of ranking points. These are awarded for winning tournaments and are what decide the ATP rankings. The graph below shows an average of the rank points held by each tournaments winner, at all all tourney levels.

![Avg. Rank of Tourney Winner]({{ site.url }}/assets/rank_graph.png)

It shows a large increase, and directly correlates with the rise of the Big Four. It's evident that in the years 2000-2004 ranking points were more evenly distrubuted, and that there was a wider range of tournaments winners.

### Returning to returns

I've often heard that the importance of serving big has been on the decline in recent years, with a bigger emphasis being placed on return of serve. This shift towards defence is a common trend amongst the top players. Although big servers like Raonic, Isner and Karlovic have had success, they haven't been able to crack top echelons of the tour, and have won no grand slam titles. To visualise this shift, below is a graph of the average number of aces that each tournament winner had throughout that tournament, prior to the final.

![Avg. Aces of Tourney Winner]({{ site.url }}/assets/aces_graph.png)

The data definitely backs up the fact that having a big serve is becoming less important when it comes to winning big tournaments. As for why, that's anyones guess. Some claim that surfaces are getting slower, but it could just be that the Big Four aren't so reliant on serving and this has shaped the number of aces.

### From the Big Four, to endurance

As I was looking through the data searching for interesting trends to explore, I thought it would be interesting to have a look at whether statistics could be used to predict the outcome of a tournament final. There wasn't much of a correlation with most categories, and most links that I found were rather obvious and uninteresting. Tournament seeding and ATP rank were the best predictors of who would win, but don't reveal much about the tournament or the player.

Something that seems to be made a big deal of in tournaments, especially Grand Slams, is how tough a journey a player has had to make before reaching the finals. It makes sense that a player that has had a slew of gruelling five setters would be physically worse off than one who has blitzed every match in straight sets.

In order to visualise this, I decided to come up with my own metric to measure the difficult of a players path to the finals. For every Grand Slam final since 2000 (French Open 2017 excluded - it happened today!), I calculated the number of minutes that both players had spent on court, not including the final itself. I averaged the figures for both winner and loser, and then subtracted the loser from the winner. The formula for this can be written as

`avg. minutes played differential = avg. winner time - avg. loser time`

A negative score in this metric indicates that the winner had a much easier run to the finals, whereas a positive score shows the winner having a more difficult one. I was quite surprised at the results, and they were far more dramatic than I expected.

As a point of reference, the average minutes played are as follows: Australian Open (879), French Open/Roland Garros (856), US Open (902), and Wimbledon (891).

![Avg. Mins of Tourney Winner]({{ site.url }}/assets/mins_graph.png)

The results show that for the Australian Open and Wimbledon, the player that had played the least minutes would on average go on to win the final. For the French Open and US open, this trend is reversed, with the player who had played for longer being more likely to go on and win.

I've got a few hypothesises as to why this is the case. The first is court speed. Wimbledon is played on grass, and the Australian is played on a special type of hard court. Grass is considered the fastest surface, and the Australian Open is often regarded as having a particularly fast hard surface. This generally favours more aggressive players, and if you're capable of hitting winners then you'll probably have a faster run to the finals.

As for the French Open, it's played on clay, which is a very slow surface. Games are usually more drawn out, and it's much harder to hit winners or aces. Following from this, a player who was more capable of enduring long games would perhaps fare better on this surface. It's also worth noting that of the 17 French Opens included in this average, Nadal won 9, and his gamestyle favours endurance.

The US Open is a bit of a mystery, as it's played on a hard court. One would expect it to be somewhere in the negative, but maybe there are outliers, or the surface is slower than the Australian Open. Regardless, these figures give a great example of how the Grand Slams all differ.

### Wrapping up

It's been a blast looking at all these statistics, and I've certainly learned more about the game that I love so much. Keep reading for a look into how I got these figures, and the tools I used to examine the data.

### CSV wrangling

The statistics I used were in comma separated value (.csv) form, and didn't require much cleaning. I wrote a program in Python combine the data and turn it into insert statements that I used to populate a PostgreSQL database.

Batch open all .csv in the current directory:
{% highlight python %}
for file in glob.glob("*.csv"):
    array = csv_to_array(file)
    for line in array:
        statements = prepare_statement(line)
        file_insert.write(statements[0])
        file_insert.write(statements[1])
{% endhighlight %}

Split the data into winner and loser, remove apostrophes, and replace empty fields with NULL. The more work you done making sure everything will work nicely in SQL, the better:
{% highlight python %}
def prepare_statement(line):
    winner_statement = ("INSERT INTO %s VALUES ('winner'," % table_name)
    loser_statement = ("INSERT INTO %s VALUES ('loser'," % table_name)
    # csv indexes for which fields to split and which are shared
    winner_fields   = (0,1,2,3,4,5,6,7,8,9,10,11,
                       12,13,14,15,16,27,28,29,30,
                       31,32,33,34,35,36,37,38,39)
    loser_fields    = (0,1,2,3,4,5,6,17,18,19,20,21,
                       22,23,24,25,26,27,28,29,30,
                       40,41,42,43,44,45,46,47,48)
    assert( len(loser_fields) == len(winner_fields) )

    # prepare fields and then create DML statement
    for i in winner_fields:
        line[i] = escape_apostrophes(line[i])
        if line[i] == 'NULL':
            entry = ( "%s" % line[i] )
        else:
            entry = ( "'%s'" % line[i] )
        if i != winner_fields[-1]:
            entry += ","
        else:
            entry += ");\n"
        winner_statement += entry

    for i in loser_fields:
        line[i] = escape_apostrophes(line[i])
        if line[i] == 'NULL':
            entry = ( "%s" % line[i] )
        else:
            entry = ( "'%s'" % line[i] )
        if i != loser_fields[-1]:
            entry += ","
        else:
            entry += ");\n"
        loser_statement += entry

    return [winner_statement, loser_statement]
{% endhighlight %}

This program produced around 26mb of beautiful INSERT INTO statements that looked like:

{% highlight sql %}

INSERT INTO matches VALUES ('winner','2000-717','Orlando','Clay','32','A','20000501','1','102179',NULL,NULL,'Antony Dupuis','R','185','FRA','27.18138261','113','351','3-6 7-6(6) 7-6(4)','3','R32','162','8','1','126','76','56','29','16','14','15');
INSERT INTO matches VALUES ('loser','2000-717','Orlando','Clay','32','A','20000501','1','102776','1',NULL,'Andrew Ilie','R','180','AUS','24.03559206','50','762','3-6 7-6(6) 7-6(4)','3','R32','162','13','4','110','59','49','31','17','4','4');

{% endhighlight %}

### All your (data)base are belong to us

The schema that I used is as follows. Just one table... It's certainly no [Boyce-Codd normal form](https://en.wikipedia.org/wiki/Boyce%E2%80%93Codd_normal_form) but it got the job done.

{% highlight sql %}
DROP TABLE matches;

CREATE TABLE matches (
    outcome VARCHAR,
    tourney_id VARCHAR,
    tourney_name VARCHAR,
    surface VARCHAR,
    draw_size INTEGER,
    tourney_level CHAR(1),
    tourney_date DATE,
    match_num INTEGER,
    player_id INTEGER,
    player_seed INTEGER,
    player_entry VARCHAR,
    player_name VARCHAR,
    player_hand CHAR(1),
    player_ht INTEGER,
    player_ioc CHAR(3),
    player_age NUMERIC,
    player_rank INTEGER,
    player_rank_points INTEGER,
    score VARCHAR,
    best_of INTEGER,
    round VARCHAR,
    minutes INTEGER,
    player_ace INTEGER,
    player_doublefault INTEGER,
    player_serve_point INTEGER,
    player_1st_in INTEGER,
    player_1st_won INTEGER,
    player_2nd_won INTEGER,
    player_serve_games INTEGER,
    player_breakpoint_saved INTEGER,
    player_breakpoint_faced INTEGER,
    PRIMARY KEY (tourney_id, player_id, match_num)
);
{% endhighlight %}

Now that I had a populated Postgres database, I did a bit of SQL magic which works out the sum of all statistics played in a tournament for each player (finals excluded), and then joins those figures with the finalists for the corresponding tournament. I created a view to house this fairly complicated statement so I could use it more easily later on. The database had around 106,000 tuples in it.

{% highlight sql %}
CREATE OR REPLACE VIEW tourney_stats AS
SELECT tourney_id, tourney_name, surface, tourney_level, player_seed, player_name, outcome, player_ht, player_ioc, player_rank, player_rank_points, player_age, score,  tourney_mins, tourney_aces, tourney_df, tourney_serve_point, tourney_1st_in,
	   tourney_1st_won, tourney_2nd_won, tourney_serve_games, tourney_bp_saved, tourney_bp_faced
FROM
        (
        SELECT tourney_id, player_name,
        SUM(minutes) as tourney_mins, SUM(player_ace) as tourney_aces,
        SUM(player_doublefault) as tourney_df,
        SUM(player_serve_point) as tourney_serve_point,
        SUM(player_1st_in) as tourney_1st_in, SUM(player_1st_won) as tourney_1st_won,
        SUM(player_2nd_won) as tourney_2nd_won, SUM(player_serve_games) as tourney_serve_games, SUM(player_breakpoint_saved) AS tourney_bp_saved,
        SUM(player_breakpoint_faced) as tourney_bp_faced
        FROM matches
        WHERE minutes IS NOT NULL
        AND round <> 'F'
        GROUP BY tourney_id, player_name
        ) AS min
JOIN
        (
        SELECT * FROM matches WHERE round = 'F'
        ) AS players
USING (tourney_id, player_name);
{% endhighlight %}

I made queries into the above view to try to find interesting trends. As an example, a query that shows the average rank for each finalist based on outcome is given by:

{% highlight sql %}
SELECT outcome, AVG(player_rank) AS avg_player_rank
FROM tourney_stats
WHERE tourney_level = 'M'
GROUP BY outcome;
{% endhighlight %}

This produces the (unsurprising) result:
{% highlight csv %}
"outcome",  "avg_player_rank"
"winner",   6.1
"loser",    14.2
{% endhighlight %}

As you can see, SQL provides a very powerful way to directly query data and produce results that would otherwise be quite difficult to find. Once I had created the view, I was able to quickly find trends with only a few declarative commands.

### Getting visual

In order to plot the figures for this article, I used a Jupyter Notebook running Python3. It was trivial to export query results as .csv and come up with quick graphs. The general formula I followed is seen below, which is for the graph showing the average age of tournament winners:

{% highlight python %}
%matplotlib inline
import pandas as pd
import matplotlib.pyplot as plt

age = pd.read_csv('age.csv')

fig = plt.figure();
age = age.set_index('player_entry')

age.plot.line(alpha=.5, stacked=True, color='g', figsize=(10, 6), legend=None)
plt.ylim([20,32])
plt.grid(alpha=.2)

plt.suptitle('Avg. Age of Tourney Winner', fontsize=20)
plt.xlabel('For Grand Slams and Masters', fontsize=14)
plt.plot([2000, 15], [2017, 32], 'k-')
plt.show()

fig.savefig('age.png')
{% endhighlight %}

### Generating this web page

In order to build this site I used the static site generator [Jekyll](https://jekyllrb.com), which makes it really easy to deploy a templated site to Github Pages, who'll host it for free. It offers nice, minimal themes (this is one is Minima), and has great native support for syntax highlighting. Click the [About]({{ site.url }}/about) button in the header (or here) to learn more about Jekyll and this theme.

I simply created a new site:

{% highlight bash %}
$ jekyll new project
{% endhighlight %}

Wrote the article using a flavour of markdown called Kramdown:

{% highlight markdown %}
### This is getting meta
{% endhighlight %}

And then created a repository to git and pushed everything there!

{% highlight bash %}
$ git push -u origin master
{% endhighlight %}

### I'm out

I've had a ball delving into tennis statistics, I hope you have too!
