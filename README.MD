# Grand Challenge of Distributed Event-Based Systems 2015

http://www.ics.uci.edu/~debs2016/call-grand-challenge.html

The ACM DEBS 2016 Grand Challenge is the sixth in a series of challenges which seek to provide a common ground and uniform evaluation criteria for a competition aimed at both research and industrial event-based systems. The goal of the 2016 DEBS Grand Challenge competition is to evaluate event-based systems for real-time analytics over high volume data streams in the context of graph models.

The underlying scenario addresses the analysis metrics for a dynamic (evolving) social-network graph. Specifically, the 2016 Grand Challenge targets following problems: (1) identification of the posts that currently trigger the most activity in the social network, and (2) identification of large communities that are currently involved in a topic. The corresponding queries require continuous analysis of a dynamic graph under the consideration of multiple streams that reflect updates to the graph.

The data for the DEBS 2016 Grand Challenge is based on the dataset provided together with the LDBC Social Network Benchmark. DEBS 2016 Grand Challenge takes up the general scenario from the 2014 SIGMOD Programming Contest, however, in contrasts to the SIGMOD contest, it explicitly focuses on processing streaming data and thus dynamic graphs. Details about the data, queries for the Grand Challenge, and information about evaluation are provided below.

## Data

Input Data Streams

The input data is organized in four separate streams, each provided as a text file. Namely, we provide the following input data files:

friendships.dat: <ts, user_id_1, user_id_2>

| Fields        | Descriptions |
| ------------- |---------------|
| ts | is the friendship's establishment timestamp |
| user_id_1 | is the id of one of the users |
| user_id_2 | is the id of the other user |

posts.dat: <ts, post_id, user_id, post, user>

| Fields        | Descriptions |
| ------------- |---------------|
| ts | is the friendship's establishment timestamp |
| post_id | is the unique id of the post |
| user_id | is the unique id of the user |
| post | is a string containing the actual post content |
| user | is a string containing the actual user name |

likes.dat: <ts, user_id, comment_id>

| Fields        | Descriptions |
| ------------- |---------------|
| ts | is the friendship's establishment timestamp |
| user_id | is the id of the user liking the comment |
| comment_id | is the id of the comment |

Please note that as the contents of files represent data streams, each file is sorted based on its respective timestamp field.

A small sample set of input data streams can be downloaded from this URL: https://www.dropbox.com/s/vo83ohrgcgfqq27/data.tar.bz2. These files are only meant for development and debugging. A bigger data set can be downloaded from here: https://www.dropbox.com/s/dcy21uc745pv98i/data.zip?dl=0. Our tests will be based on other files (about the same size as the provided larger data set), but strictly following the same format. 

## Query 1

The goal of query 1 is to compute the top-3 scoring active posts, producing an updated result every time they change.

The total score of an active post P is computed as the sum of its own score plus the score of all its related comments. Active posts having the same total score should be ranked based on their timestamps (in descending order), and if their timestamps are also the same, they should be ranked based on the timestamps of their last received related comments (in descending order). A comment C is related to a post P if it is a direct reply to P or if the chain of C's preceding messages links back to P.

Each new post has an initial own score of 10 which decreases by 1 each time another 24 hours elapse since the post's creation. Each new comment's score is also initially set to 10 and decreases by 1 in the same way (every 24 hours since the comment's creation). Both post and comment scores are non-negative numbers, that is, they cannot drop below zero. A post is considered no longer active (that is, no longer part of the present and future analysis) as soon as its total score reaches zero, even if it receives additional comments in the future.

Input Streams: posts, comments
Output specification:

<ts,top1_post_id,top1_post_user,top1_post_score,top1_post_commenters,
top2_post_id,top2_post_user,top2_post_score,top2_post_commenters,
top3_post_id,top3_post_user,top3_post_score,top3_post_commenters>

ts: the timestamp of the tuple event that triggers a change in the top-3 scoring active posts appearing in the rest of the tuple
topX_post_id: the unique id of the top-X post
topX_post_user: the user author of top-X post
topX_post_commenters: the number of commenters (excluding the post author) for the top-X post

Results should be sorted by their timestamp field. The character "-" (a minus sign without the quotations) should be used for each of the fields (post id, post user, post commenters) of any of the top-3 positions that has not been defined. Needless to say, the logical time of the query advances based on the timestamps of the input tuples, not the system clock.
Sample output tuples for the query

2010-09-19 12:33:01.923+0000,25769805561,Karl Fischer,115,10,25769805933,Chong Liu,83,4,-,-,-,-
2010-10-09 21:55:24.943+0000,34359739095,Karl Fischer,58,7,34359740594,Paul Becker,40,2,34359740220,Chong Zhang,10,0
2010-12-27 22:11:54.953+0000,42949673675,Anson Chen,127,12,42949673684,Yahya Abdallahi,69,8,42949674571,Alim Guliyev,10,0 

## Query 2

This query addresses the change of interests with large communities. It represents a version of query type 2 from the 2014 SIGMOD Programming contest. Unlike in the SIGMOD problem, the version for the DEBS Grand Challenge focuses on the dynamic change of query results over time, i.e., calls for a continuous evaluation of the results.

Goal of the query:
Given an integer k and a duration d (in seconds), find the k comments with the largest range, where the range of a comment is defined as the size of the largest connected component in the graph defined by persons who (a) have liked that comment (see likes, comments), (b) where the comment was created not more than d seconds ago, and (c) know each other (see friendships).

Parameters: k, d

Input Streams: likes, friendships, comments

Output:
The output includes a single timestamp ts and exactly k strings per line. The timestamp and the strings should be separated by commas. The k strings represent comments, ordered by range from largest to smallest, with ties broken by lexicographical ordering (ascending). The k strings and the corresponding timestamp must be printed only when some input triggers a change of the output, as defined above. If less than k strings can be determined, the character “-” (a minus sign without the quotations) should be printed in place of each missing string.

The field ts corresponds to the timestamp of the input data item that triggered an output update. For instance, a new friendship relation may change the size of a community with a shared interest and hence may change the k strings. The timestamp of the event denoting the added friendship relation is then the timestamp ts for that line's output. Also, the output must be updated when the results change due to the progress of time, e.g., when a comment is older that d. Specifically, if the update is triggered by an event leaving a time window at t2 (i.e., t2 = timestamp of the event + window size), the timestamp for the update is t2. As in Query 1, it is needless to say that timestamps refer to the logical time of the input data streams, rather than on the system clock.

In summary, the output is specified as follows:

ts: the timestamp of thetuple event that triggers a change in the output.
comments_1,...,comment_k: top k comments ordered by range, starting with the largest range (comment_1).
Sample output tuples for the query with k=3 could look as follows:

2010-10-28T05:01:31.022+0000,I love strawberries,-,-
2010-10-28T05:01:31.024+0000,I love strawberries,what a day!,-
2010-10-28T05:01:31.027+0000,I love strawberries,what a day!,well done
2010-10-28T05:01:31.032+0000,what a day!,I love strawberries,well done