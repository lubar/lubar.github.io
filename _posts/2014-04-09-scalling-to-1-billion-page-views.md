---
layout: post
category : lessons
tagline: "Supporting tagline"
tags : [tomnod, AWS, MH370, tutorial]
---
{% include JB/setup %}

3 Tips for Scaling your Website
to 1 Billion Page Views
=====================================================

Last week was a time of stress, uncertainty and emotion as millions of people around the world searched and waited for any news about the location of the missing Malaysia Airlines flight 370 that disappeared on the evening of Friday March 7. It was also a week of stress, uncertainty and emotion for the Tomnod developers as we scrambled to support millions of people from around the world searching the possible crash location on Tomnod.com. 

Our crowdsourcing campaign inviting the public to search DigitalGlobe satellite images and help identify clues about the plane’s whereabouts received a phenomenal response. In the first 24 hours, we received half a million unique users and recorded 25M page views. Over the week, traffic grew to 1 billion views as the crowd searched xxTB of high-resolution DigitalGlobe imagery, covering 50,000km2 of the surface of the earth. We were featured on the front page of Reddit, CNN, Fox News and other media and, best of all, we got thousands of emails, tweets, Facebook messages and phone calls from our amazing crowd.
[graph of traffic, page views, or similar, over time]

The story about the crash [has now come out / may never come out] but the story of how we were able to scale from a site that usually received 500 users / day to over 1 million unique visitors every day can be summarized in three simple lessons.

Our app architecture is fairly simple: web and API servers load static (satellite) image content and read/write (dynamic) data to a SQL database.

Lesson #1 – Cache your Content
-------------------------------
Tomnod relies on a lot of content. Specifically, our high-resolution images from DigitalGlobe satellites are stored as map-tile-pyramids[link]: a stack of 256x256 jpeg tiles that each correspond to about 40m2 on the surface of the earth. With over 50,000km2 of imagery, we were hosting over a billion images.

Our tiles come from DigitalGlobe Cloud Services[link]: an online delivery platform for all DigitalGlobe’s imagery (think Google Maps; but with hundreds of images at every point on the earth, capturing the views of the planet from the past 10 years). Once a Tomnod user requests a tile from DGCS, we cache that tile on Amazon S3 for future users. We wrote our own cache to A) check if the tile is in the S3 cache and, if not, B) fetch it from DGCS and then C) PUT it in the cache. A-C can take a few seconds – normally no problem but, when each tile is suddenly being requested by 500 users simultaneously, we get 500 attempts to fetch from DGCS and 500 PUTs to S3 … for each of our 1 billion tiles!

DGCS was smart and throttled our access so that we didn’t bring down the service for other DigitalGlobe customers. But then we started getting [504] errors from S3 telling us to slow down with our PUT and GET requests! The AWS team gave us a call and recommended two fixes: First, we hosted our S3 bucket on CloudFront, Amazon’s global content delivery network. Second, AWS modified our hardware architecture: normally each bucket lives on a single (or few) physical drives so that thousands of parallel reads or write become a challenge. By spreading out bucket out across multiple drives, we were able to increase our PUT speeds and scale up our GETs. Finally, we re-wrote our caching protocol to pre-fetch every tile from DGCS only once.

Result: no more cache misses, no more delays in loading image tiles and deliver imagery to 1M users all over the globe.

Lesson #2 – Pool your Connections
---------------------------------
Once we figured out the challenge of getting imagery in front of our users, people started viewing, searching and tagging every single pixel, hundreds of times over. Each of these interactions had to sync over our API to our Postgresql database (running the PostGIS geospatial extension), resulting in tens of thousands of reads and writes per second. Each API hit created a new connection to the database, did its thing, closed the connection and sent data back to the client. This meant we were trying to open thousands of connections every second and we rapidly exceeded our connection pool, resulting in no data and site failure. 

Our first response was to put up a simple error message so that people would A) realize that there was a problem and B) stop hitting refresh! And so it was that the “Frown Clown” was born [IMG]. 

Frown Clown was a sad solution to a simple problem: how do we manage the number of database connections more efficiently? One idea was to keep a persistent connection open for each connected client but that would have required persistent state management and some fairly major code rewrites. A better solution came in the form of PG-Bouncer [link].

PG-Bouncer sits in front of your Postgres database server and collects all incoming connections. When it makes sense, it pools queries into a single connection, executes the SQL and returns all results before closing the connection. Now, instead of 1,000 queries requiring 1,000 connection, they can be accomplished in just one – all without any noticeable delay to the end user. Setup was simple and tuning PG-Bouncer for our app was quick and easy.

Result: pooling database connections with PG-Bouncer vastly increased our ability to handle simultaneous API hits and turned the frown clown upside-down!

Lesson #3 – Cache your Queries
------------------------------
Now that we could handle thousands of connected users, they started running tens of thousands database queries each second. In many cases, these queries were identical for many users (e.g., tell me what we’re looking for) or repeated very regularly (e.g., tell me where to look). We always knew it was a smart idea to cache the results of these queries in memory, rather than computing them from the database… we just weren’t smart about how we did it! 

We were running memcache in front of our API servers: but this meant that each web server had its own instance of memcache. Once the ELB scaled up to 100 web servers, we had 100 caches, each of which had to be filled by hitting the database. This effectively lost the benefit of having a cache at all.

The solution was to setup a dedicated (set of) memcache server that could buffer between the database and ALL the API servers. Now the web servers could scale infinitely and we would preserve the same cache for all our users’ common queries.

Result: with a bit of hacking and a bit of tuning, a dedicated memcache server decreased the load on the database server from 100%+ to about 10-20%, even with tens of thousands of simultaneous users.
