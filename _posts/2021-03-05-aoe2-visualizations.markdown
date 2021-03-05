---
layout: single
title:  "Age Of Empires II: Visualizations Part I"
author_profile: true
classes: wide
---
(This is going to be a multi-post blog because that might make things more manageable for me. Also, now I can have a long winded introduction and not feel bad about it. And I won't be hiding my eventual graphics below this wall of text.)

# Introduction to a 20+ year old game

Age of Empires II is a RTS (Real Time Strategy) game that came out in 1999. But despite this game being over 20 years old, it has a fairly large player base to this day. In the last couple years, a new remaster of the game was released, AOE2 Definitive Edition. This greatly enhanced the graphics along with small edits.

My history with AOE2 goes back to when I was a kid and my brother bought the game. Back then I thought it was the coolest game even though I struggled pretty greatly due to a real lack of literacy.

But many years ago I had mostly forgotten about the game, there was a brief moment I played with a couple friends my freshman year of college but that quickly fizzled out.


More recently, I happened upon some videos of people playing competitively. I was pretty shocked to see that there were plenty of people still interested in this game. After watching some videos and streams of competitive matches, I decided I wanted to give a try at the game again. This time though I was much better prepared (being above the age of 10 helps a lot).

After practicing a bit against the computer and reading up on build orders and other tips I decided to dip into the ranked ladder.

AOE2:DE uses an [Elo system][elo] to rank players and try to mat together with similar skill. Elo gives you a certain number of points, winning increases your points, losing decreases your points. There are some cool websites and tools people have made that track some of the data that you can pull from all of this. The most popular is probably [aoe2.net](https://aoe2.net/). This website tracks all the online matches, and keeps track of all the wins/losses for a given person.

![aoe2.net plot](/assets/aoe2_net_plot.png)

Above is my Elo ranking over time. Initially I started at ~800-900 elo and quickly found that those people were too good for me and dropped down. I don't play too often so I stayed pretty low until a more recent resurgence. Anyways, looking at this I thought I could do a lot better. The time axis really makes things sloppy once I play a couple or more games in a single day. Additionally there is a lot more information than just my Elo that I wanted to track and look into. So this is my path to doing just that.


# Scraping That Data
Luckily, aoe2.net has a pretty friendly API that I could use to scrape all the data from my matches. The bright side is it returns basically anything I would want to know about a match, and does this for every single match I've played up to that point.

The downside is the format it returns this data is a bit of a slog, this is the string of data for just 2 games (yes that is a single line of text).

{% highlight text %}
[{"match_id":"72630745","lobby_id":null,"match_uuid":"0283ffe0-051d-0c4a-986f-1c489648336d","version":"45340","name":"AUTOMATCH","num_players":2,"num_slots":2,"average_rating":null,"cheats":false,"full_tech_tree":false,"ending_age":5,"expansion":null,"game_type":0,"has_custom_content":null,"has_password":true,"lock_speed":true,"lock_teams":true,"map_size":0,"map_type":17,"pop":200,"ranked":true,"leaderboard_id":3,"rating_type":2,"resources":1,"rms":null,"scenario":null,"server":"ukwest","shared_exploration":false,"speed":2,"starting_age":2,"team_together":true,"team_positions":true,"treaty_length":0,"turbo":false,"victory":1,"victory_time":0,"visibility":0,"opened":1614038936,"started":1614038936,"finished":1614040506,"players":[{"profile_id":3238633,"steam_id":"76561198202213716","name":"Super Spooper","clan":null,"country":"US","slot":1,"slot_type":1,"rating":869,"rating_change":16,"games":null,"wins":null,"streak":null,"drops":null,"color":2,"team":1,"civ":26,"won":true},{"profile_id":3399222,"steam_id":"76561198032315208","name":"King-Uter","clan":null,"country":"DE","slot":2,"slot_type":1,"rating":876,"rating_change":-16,"games":null,"wins":null,"streak":null,"drops":null,"color":1,"team":2,"civ":6,"won":false}]},{"match_id":"72627884","lobby_id":null,"match_uuid":"b168c07d-2d9a-f44a-8dae-530a479d23a1","version":"45340","name":"AUTOMATCH","num_players":2,"num_slots":2,"average_rating":null,"cheats":false,"full_tech_tree":false,"ending_age":5,"expansion":null,"game_type":0,"has_custom_content":null,"has_password":true,"lock_speed":true,"lock_teams":true,"map_size":0,"map_type":9,"pop":200,"ranked":true,"leaderboard_id":3,"rating_type":2,"resources":1,"rms":null,"scenario":null,"server":"eastus","shared_exploration":false,"speed":2,"starting_age":2,"team_together":true,"team_positions":true,"treaty_length":0,"turbo":false,"victory":1,"victory_time":0,"visibility":0,"opened":1614037629,"started":1614037629,"finished":1614038767,"players":[{"profile_id":4411548,"steam_id":"76561198269088685","name":"Regax","clan":null,"country":"CA","slot":1,"slot_type":1,"rating":null,"rating_change":null,"games":null,"wins":null,"streak":null,"drops":null,"color":2,"team":1,"civ":24,"won":false},{"profile_id":3238633,"steam_id":"76561198202213716","name":"Super Spooper","clan":null,"country":"US","slot":2,"slot_type":1,"rating":854,"rating_change":15,"games":null,"wins":null,"streak":null,"drops":null,"color":1,"team":2,"civ":30,"won":true}]}]`
{% endhighlight %}

Now I played ~60 games as I'm working on this, so that gets out of hand pretty quickly. The real goal is to turn this into a [pandas dataframe](https://pandas.pydata.org/) that would be a lot easier to work with.

First step is getting this data locally, which a quick Curl command does nicely, just have to supply my steam id and the number of games I want to pull (setting a higher number works just fine)

{% highlight terminal %}
$ curl "https://aoe2.net/api/player/matches?game=aoe2de&steam_id=76561198202213716&count=500" -o data.txt" -o data.txt
{% endhighlight %}

this saves the data to a simple text file which I can now open using python and parse the crap out of it.

Since each match string starts with `{"match_id":` we can split on that, then tack that back on and trim off some useful end brackets/commas.

{% highlight python %}
with open(filename, 'r') as f:
    game_list=f.readline().split('{"match_id":')

    #clean list
    game_list=game_list[1:]
    for i in range(len(game_list)):
        game_list[i] = '"match_id":%s'%game_list[i][:-2]
{% endhighlight %}

So now we have a list, `game_list` where each element is the string of data associated with a single game. These strings includes info about the game (e.g. name of map, time it started, average elo of players) as well as more player specific info (e.g. what is their elo, did they win, etc.).

# Creating Pandas
Now to convert these strings into something I can load into pandas. For each game I can split the string into `game_data` and `player_data`. `player_data` is repeated twice since there's two players per match. To sepearte the two:

{% highlight python %}
gdata, combined_pdata= game_list[i].split('"players"')
gdata= gdata[:-1].split(',')
p1data, p2data = combined_pdata[3:-2].split('},{')
{% endhighlight %}

Now we have lists `gdata`, `p1data`, and `p2data` which are of the form `["some_category:some_data", ...]`

Finally we just remove the `"some_category:"` part of the list and we have our data! and simply importing into a Pandas dataframe gives us what we want.

![image of pandas df](/assets/aoe2_panda_df.png)
So what we leave off now with is a pandas dataframe with 75 total columns, and a bunch a bunch of `null` values because most of the info doesn't get captured for some reason. So we'll have to narrow down and focus on the most interesting categories as well as fill in some info (ie sometimes average Elo is a null value but we can look at elo of both players quite easily).

Next time: making some figures, learning to deal with incomplete data.

[elo]: https://en.wikipedia.org/wiki/Elo_rating_system
