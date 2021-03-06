REVERBERATIONS
==============
 
SwiftRiver is a FOSS platform for determining the veracity and accuracy of news related to an event or topic. Swift has an extremely modular design with support for plug-ins and hosted API services.  Reverberations is one of those API services for determining the influence of content across the social web. You can find out more about Swift Web Services at - http://swiftly.org

	Find us on Github - http://github.com/appfrica/reverberations

Change Log
----------

* V0.1.0: Initial stable release of the software.


Functionality
-------------

Given a tweet ID (must be originating tweet ID, no retweets) this script will create a tree mapping the retweets, along with a corresponding rank value for that tweet/retweet. The tree is returned as an array of nodes, with each node holding information about the retweet, and the parent retweet/tweet.


Installation
------------

1. In the directory where the script is saved type:
	python twitter_reverb.py <tweetID>
	<tweetID> is the unique tweetID of the originating tweet
2. If this is the first time using the script, it will prompt you to authenticate by providing you a link that you can follow to authenticate with twitter directly.
3. Login with your twitter account and hit "allow".
4. Twitter will load a page with a pin number, copy this pin number and paste it into the command line window where it says "Please enter the pin code"
5. The script will save the "token" and the "secret" for use in future sessions, in a file named token.txt (currently this does not work, you will have to re-authenticate each time you use this script).
6. A python pickle file named "jsonRanks.txt" will be created to hold the information of the resultant retweet tree, including all information associated with each node in the tree. The node structure described below in the section "Information Structures"
7. To unpickle this file and create a python object simply call "pickle.load(file)" where "file" is an opened fill stream.


Information Structures
----------------------

The tree is returned as an array of nodes. Each node in the tree will have information on the specific tweet, with the top node in the tree being the originating tweet. The information included in each node is: 

	user = user object
	rank = the relevance rank of this tweet/retweet
	lvl =  the level in the tree that this node is located (0 being the 		originating tweet level)
	ts = time stamp at which the tweet or retweet was created.
	twid =  the tweet id of this specific tweet
	rtusr = the parent tweet id, the tweet that tweet is a rewet of

Example on how to access node information:

	lets say there is a node object named tweet

	handle of tweet = "tweet.user.screen_name"
	the id of the user = "tweet.user.id"
	rank of retweet = "tweet.rank"
	level of retweet in tree = "tweet.lvl"
	time stamp of retweet = "tweet.ts"
	tweet id = "tweet.twid"
	parent tweet = "tweet.rtuser"


Ranking
-------

Each retweet is given a rank value based on the number of direct retweets it creates, as well as a weighted value for each successive retweet of a retweet, based on the level distance between the current tweet and the children. This is not a ranking of the nodes, it is simply a value for describing the influence a node has, the higher the value, the more influence that the node had. For example:

Lets say there tree looks like this...

		   top node
	     	/  	 \
	      rt1	 rt2
	     /	 \	/   \
	   rt3	 rt4  rt5   

1) rt3, rt4, rt5, would all have a rank of 0 
	b/c they have no child retweets.
	
2) rt1 has a rank of 1
	b/c it has 2 children retweets
	
3) rt2 has a rank of 1
	b/c it has exactly one direct child retweet
	
4) top_node would have a rank of 3.5 b/c (1) it has 2 direct children retweets so that is 2 pts counted towards the ranking value (2) it has .5 points for each of the retweets rt3, rt4, and rt5, this is b/c each of these retweets are 	a distance of 2 away from the top node, so they are only work 1/2 pts. If a node were a distance of 3 away it would be worth 1/3 pts


Tree Formation
--------------

Since the introduction of the new twitter retweet style, it is not possible to get exact information on whether a retweet is a retweet of the original tweet or a retweet of another retweet. Twitter does not provide functionality via the API for obtaining this type of information. All retweets are returned as retweets of the original tweet. So in order to form the tree we make use of a couple sources of information. (please note that this does not form a tree that is always exactly as how the retweets formed. It is an extrapolation of the information and may be slightly incorrect in some cases.)
the info used include:

1. the returned tweets from the retweets API function
2. the tweet/retweet creation timestamp
3. the followers of the tweet/retweet author
	
The retweets API status function returns up to 100 of the most recent retweets of a tweet. We then use timestamp information as well as followers information gathered via the API to create the tree, the logic is as follows.

1. A retweet cannot have been created before its parent tweet was created, so a tweet/retweet with a timestamp prior to another retweet can never be the second retweets child retweet.
2. if a person is following someone, then he most likely retweet that person tweet/retweet, while also satisfying condition 1.
3. if a person is following 2 people that retweeted a tweet, and above conditions hold. Then he most likely retweeted the tweet/retweet of the tweet with the earlier timestamp, because only the first tweet/retweet will show up in the users home.
4. if there are retweets left over after checking the above conditions on all the tweets in the retweet list. Then these must be children of retweets tweeted by people with private profiles, or of people who deleted their retweets/tweets.
5. we take the left over retweets, and try to create subtrees from these, with the top node of the subtree a direct child of the original tweet.
	
	
PROJECT CONTRIBUTORS
--------------------

Jon Gosier
Mang-Git 


	
