---
title: "Basic player EPA in the 2018/19 Premier League"
author: "@wiscostretford"
date: "2019-04-21"
output: 
  html_document:
    keep_md: yes
---

(See also my [full blog set here](https://wiscostret.wordpress.com/))

**Introduction**

Last time out, I [wrote a bit](https://wiscostret.wordpress.com/2019/04/20/win-probability-expected-points-and-epa-player-value-in-football/) on expectd points and player value with EPA ('expected points added'). 

*Expected points added* (EPA) is a player value measure that is based on the contribution made by the player to the expected points of the player's team. In short, EPA is simply the change in xPts resulting from a match event, for instance a shot. It's a measure that can be simple to understand and simple to apply as a player value concept, but which can also be scaled up in complexity, depending on the underlying 'expected points' model.

In this post, I'll review two super basic but, in my view, quite illustrative methods for assessing player value. I will take the 2018/19 Premier League as an example.

First, I will assess EPA player value based on actual scoring. Simply, which players have contributed to their team's expected points throughout the season? And second, I will assess EPA player value based on *expected* scoring, through expected goals (xG) and expected assists (xA).

To illustrate: Let's say the actual (or xG) score in the 41st minute of a game is 2 to 1. Applying a Poisson distribution, the basic match outcome probabilities at that point are 59% for a home win, 21% for a draw, and 19% for an away win. Consequently, the home team xPts is 1.98. If the home team then in the 42nd minute score a goal (or have a shot goal valued at 1.0 xG), the score is now 3 to 1. Now the match outcome probabilities are 76%, 13% and 8%, and the home team xPts is 2.41. The EPA of that goal/shot is then 2.41 minus 1.98 = 0.43.

&nbsp;

**Data**

Again I will I will use data from understat, which provides basic info for all Premier League matches, as well as (and importantly for the purposes at hand) detailed data for all shot events in big leagues matches. Here I will take the example of all Premier League games up to an including gameweek 34.

The data set, which I have manipulated slightly for the purposes at hand, looks like so:




```r
head(usdata)
```

```
##   X.1     id minute      result    X    Y         xG         player h_a
## 1   1 232811      1 BlockedShot 86.3 71.1 0.03996211 Alexis Sánchez   h
## 2   2 232812      2        Goal 88.5 50.0 0.76116884     Paul Pogba   h
## 3   3 232818     39   SavedShot 72.4 65.5 0.01839577     Paul Pogba   h
## 4   4 232819     40   SavedShot 88.0 65.3 0.08121494      Luke Shaw   h
## 5   5 232821     55   SavedShot 78.1 33.0 0.02830883 Matteo Darmian   h
## 6   6 232822     64 MissedShots 81.3 47.6 0.07612572      Juan Mata   h
##   player_id situation season  shotType match_id            h_team
## 1       498  OpenPlay   2018 RightFoot     9197 Manchester United
## 2      1740   Penalty   2018 RightFoot     9197 Manchester United
## 3      1740  OpenPlay   2018 RightFoot     9197 Manchester United
## 4      1006  OpenPlay   2018 RightFoot     9197 Manchester United
## 5       557  OpenPlay   2018 RightFoot     9197 Manchester United
## 6       554  OpenPlay   2018  LeftFoot     9197 Manchester United
##      a_team h_goals a_goals                date player_assisted lastAction
## 1 Leicester       2       1 2018-08-10 22:00:00       Luke Shaw       Pass
## 2 Leicester       2       1 2018-08-10 22:00:00            <NA>   Standard
## 3 Leicester       2       1 2018-08-10 22:00:00  Alexis Sánchez       Pass
## 4 Leicester       2       1 2018-08-10 22:00:00       Juan Mata    Chipped
## 5 Leicester       2       1 2018-08-10 22:00:00  Alexis Sánchez       Pass
## 6 Leicester       2       1 2018-08-10 22:00:00  Alexis Sánchez       Pass
##   id.1 isResult            datetime h.id           h.title h.short_title
## 1 9197     TRUE 2018-08-10 22:00:00   89 Manchester United           MUN
## 2 9197     TRUE 2018-08-10 22:00:00   89 Manchester United           MUN
## 3 9197     TRUE 2018-08-10 22:00:00   89 Manchester United           MUN
## 4 9197     TRUE 2018-08-10 22:00:00   89 Manchester United           MUN
## 5 9197     TRUE 2018-08-10 22:00:00   89 Manchester United           MUN
## 6 9197     TRUE 2018-08-10 22:00:00   89 Manchester United           MUN
##   a.id   a.title a.short_title goals.h goals.a   xG.h    xG.a forecast.w
## 1   75 Leicester           LEI       2       1 1.5137 1.73813     0.2812
## 2   75 Leicester           LEI       2       1 1.5137 1.73813     0.2812
## 3   75 Leicester           LEI       2       1 1.5137 1.73813     0.2812
## 4   75 Leicester           LEI       2       1 1.5137 1.73813     0.2812
## 5   75 Leicester           LEI       2       1 1.5137 1.73813     0.2812
## 6   75 Leicester           LEI       2       1 1.5137 1.73813     0.2812
##   forecast.d forecast.l              team   X2   Y2 result2 goals
## 1     0.3275     0.3913 Manchester United 86.3 71.1  nogoal     0
## 2     0.3275     0.3913 Manchester United 88.5 50.0    goal     1
## 3     0.3275     0.3913 Manchester United 72.4 65.5  nogoal     0
## 4     0.3275     0.3913 Manchester United 88.0 65.3  nogoal     0
## 5     0.3275     0.3913 Manchester United 78.1 33.0  nogoal     0
## 6     0.3275     0.3913 Manchester United 81.3 47.6  nogoal     0
```

&nbsp;

**Basic EPA from actual scoring**

As set out in my previous blog post, we can calculate a 'snapshot' of expected outcomes for each game event using a basic Poisson distribution. For ease, I've set up a simple function called 'poissonwp' that calculates the Poisson distribution for all possible scoring outcomes (from 0-10), based on the score inputs, and then sums up the probabilities for home win, away win, and draw. 

We can feed this to the actual scoring for all events, and calculate the expected points at the time of the goal for each team, as follows:




```r
library(dplyr)

usdata2 <- usdata %>% 

# generating a cumulative goal score for each game, and selecting our columns of interest
  group_by(match_id) %>% 
  arrange(match_id,minute) %>% 
  mutate(hgoalcum = cumsum(ifelse(h_a=="h" & result=="Goal" | h_a=="a" & result=="OwnGoal",goals,0))) %>% 
  mutate(agoalcum = cumsum(ifelse(h_a=="a" & result=="Goal" | h_a=="h" & result=="OwnGoal",goals,0))) %>% 
  select(minute,xG,player,match_id,team,h_team,a_team,goals,hgoalcum,agoalcum,result2) %>% 
  ungroup()

# calculating the poisson expected outcomes following each score event
for (i in 1:nrow(usdata2)){
  wps <- poissonwp(usdata2$hgoalcum[i],usdata2$agoalcum[i])
  usdata2$hxpo[i] <- wps[1]
  usdata2$axpo[i] <- wps[2]
  usdata2$dxpo[i] <- wps[3]
}

# converting to expected points
usdata2 <- usdata2 %>% 
  mutate(hxpts = hxpo*3 + dxpo*1) %>% 
  mutate(axpts = axpo*3 + dxpo*1)
  
# creating the epa variable

usdata2 <- usdata2 %>% 
  group_by(match_id) %>% 
  arrange(match_id,minute) %>% 
  mutate(epa = case_when(team == h_team ~ hxpts - lag(hxpts,first(hxpts)), team == a_team ~ axpts - lag(axpts,first(axpts))))
```

Now that we have constructed the player level EPA data set, we can summarize EPA and xG (for goal events only) at the player level, to see who are the 'best' players in the Premier League this season in terms of adding to their team's expected points through goals:


```r
usdata2 %>%
  filter(result2=="goal") %>% 
  group_by(player) %>% 
  summarize(goalepa = sum(epa,na.rm=T), goalxG = sum(xG)) %>% 
  arrange(-goalepa) %>%
  slice(1:20)
```

```
## # A tibble: 20 x 3
##    player                    goalepa goalxG
##    <fct>                       <dbl>  <dbl>
##  1 Mohamed Salah               15.0    8.60
##  2 Eden Hazard                 12.6    6.51
##  3 Harry Kane                  12.6    6.85
##  4 Sadio Mané                  11.9    6.48
##  5 Jamie Vardy                 11.5    7.95
##  6 Raheem Sterling             11.3    8.37
##  7 Glenn Murray                11.1    5.08
##  8 Pierre-Emerick Aubameyang   11.1    6.79
##  9 Sergio Agüero               10.2    8.16
## 10 Luka Milivojevic            10.2    7.78
## 11 Alexandre Lacazette          9.81   3.11
## 12 Callum Wilson                9.66   4.29
## 13 Richarlison                  9.46   5.13
## 14 Paul Pogba                   9.42   7.27
## 15 Raúl Jiménez                 9.39   4.63
## 16 Romelu Lukaku                8.89   4.53
## 17 Salomón Rondón               8.37   3.13
## 18 Chris Wood                   8.03   3.91
## 19 Gylfi Sigurdsson             7.96   3.44
## 20 Son Heung-Min                7.92   3.51
```

What this tells us is that Mohamed Salah has added almost 15 points to Liverpool's expected points total based on actual goals contributed. He is followed by Eden Hazard, Harry Kane and Sadio Mané.

Of note, these rankings do not simply reflect xG or goals scored. There is divergence, most importantly where goals occur when the team is leading big. In those cases, the EPA will be low, since it doesn't add much to expected points. This is one strength of EPA over raw xG as a player performance evaluator. It favours players who uniquely contribute to team expected points, scoring important goals, also 'lesser' goalscorers from smaller teams, such as Glenn Murray and Luka Milivojevic.

We can plot this against total xG to see the correlation and differences:


```r
library(ggplot2)
```

```
## Warning: package 'ggplot2' was built under R version 3.4.4
```

```r
library(ggrepel)
```

```
## Warning: package 'ggrepel' was built under R version 3.4.3
```

```r
usdata2 %>% 
  group_by(player) %>% 
  summarize(totalxG = sum(xG), goalepa = sum(ifelse(result2=="goal",epa,0),na.rm=T)) %>% 
    ggplot(aes(x=totalxG,y=goalepa)) +
      geom_point() + 
      geom_text_repel(aes(label=ifelse(goalepa>8 | totalxG>10,as.character(player),"")),size=4.5) + 
      theme_bw() +
      labs(x="Total xG",y="Goal EPA",title="Goal EPA and Total xG in the 2018/19 Premier League")
```

![](https://i.imgur.com/vRrBjaP.png)<!-- -->

&nbsp;

**Basic EPA from xG**

Now, we can do the exact same EPA player value calculation using xG only, rather than actual scoring:


```r
usdata3 <- usdata %>% 

# generating a cumulative xG score for each game, and selecting our columns of interest
  group_by(match_id) %>% 
  arrange(match_id,minute) %>% 
  mutate(hcumxG = cumsum(ifelse(h_a=="h",xG,0))) %>%
  mutate(acumxG = cumsum(ifelse(h_a=="a",xG,0))) %>% 
  select(minute,xG,player,match_id,team,h_team,a_team,goals,hcumxG,acumxG) %>% 
  ungroup()

# calculating the poisson expected outcomes following each score event
for (i in 1:nrow(usdata3)){
  wps <- poissonwp(usdata3$hcumxG[i],usdata3$acumxG[i])
  usdata3$hxpo[i] <- wps[1]
  usdata3$axpo[i] <- wps[2]
  usdata3$dxpo[i] <- wps[3]
}

# converting to expected points
usdata3 <- usdata3 %>% 
  mutate(hxpts = hxpo*3 + dxpo*1) %>% 
  mutate(axpts = axpo*3 + dxpo*1)
  
# creating the epa variable

usdata3 <- usdata3 %>% 
  group_by(match_id) %>% 
  arrange(match_id,minute) %>% 
  mutate(epa = case_when(team == h_team ~ hxpts - lag(hxpts,first(hxpts)), team == a_team ~ axpts - lag(axpts,first(axpts))))
```


```r
usdata3 %>%
  group_by(player) %>% 
  summarize(xgepa = sum(epa,na.rm=T)) %>% 
  arrange(-xgepa) %>%
  slice(1:20)
```

```
## # A tibble: 20 x 2
##    player                    xgepa
##    <fct>                     <dbl>
##  1 Mohamed Salah             13.7 
##  2 Pierre-Emerick Aubameyang 11.8 
##  3 Harry Kane                 7.74
##  4 Alexandre Lacazette        7.59
##  5 Sergio Agüero              7.39
##  6 Eden Hazard                6.81
##  7 Roberto Firmino            6.7 
##  8 Callum Wilson              6.38
##  9 Sadio Mané                 6.35
## 10 Paul Pogba                 6.23
## 11 Raúl Jiménez               5.9 
## 12 Romelu Lukaku              5.79
## 13 Ashley Barnes              5.70
## 14 Danny Ings                 5.68
## 15 Gylfi Sigurdsson           5.59
## 16 David Silva                5.54
## 17 Richarlison                5.48
## 18 Jamie Vardy                5.34
## 19 Raheem Sterling            4.91
## 20 Gabriel Jesus              4.56
```

What this tells us is that Mohamed Salah, again, comes out on top, adding almost 14 points to Liverpool's expected points total - if each game went as per xG (rather than actual scoring). However, there is some shuffling around at the top, and especially we see that important players on small teams, like Glenn Murray who featured above, fall out, given that their sporadically crucial but unexpected (statistically) goals do not feature heavily in xG.

Like above, there will inevitably be some divergence here, given xG does not exactly correlate with actual outcomes. Where players miss high-xG chances, this will add to the team's xG-based expected points (and the player thus credited with a high EPA), when in fact this is not the case. However, the idea of using xG is that it is a 'truer' reflection of player performance, and we know that in the long-term it correlates well with actual performance.

Again to spot correlation and differences, we can plot the xG-based EPA against total xG:


```r
library(ggplot2)
library(ggrepel)

usdata3 %>% 
  group_by(player) %>% 
  summarize(totalxG = sum(xG), xgepa = sum(epa,na.rm=T)) %>% 
    ggplot(aes(x=totalxG,y=xgepa)) +
      geom_point() + 
      geom_text_repel(aes(label=ifelse(xgepa>8 | totalxG>10,as.character(player),"")),size=4) + 
      theme_bw() +
      labs(x="Total xG",y="xG EPA",title="xG EPA and Total xG in the 2018/19 Premier League")
```

![](https://i.imgur.com/zZLjK72.png)<!-- -->

**Conclusion**

This concludes my second look (see [part 1 here](https://wiscostret.wordpress.com/2019/04/20/win-probability-expected-points-and-epa-player-value-in-football/)) at player value using EPA.

I hope now to have shown that these concepts provide much promise for football analytics, as relatively simple ideas that can be leveraged for advanced analysis. This is now well-established in other sports, but football could benefit as well.

What is moreover important to stress is that what I have gone through here are the most basic of EPA models. These could be improved massively by looking at other important variables influencing win probability and expected points, as well as individual player contributions to these. That's, I hope, a next step (though maybe not for me).

Comments and feedback are welcome. Any questions, go find me on Twitter [wiscostretford](http://www.twitter.com/wiscostretford)
