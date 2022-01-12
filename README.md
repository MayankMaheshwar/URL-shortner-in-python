# URL-shortner-in-python

Usage -
Run the api.py program and give the url you wanted to short or retrieve original_url


# Requirements of the System
The URL shortening service like Tiny-URL should have the following features:

# Functional Requirements

Users should be able to generate a shortened URL from the original URL.
The short link when given should redirect users to the original link.


# Non-Functional Requirements
The non-functional requirements include:
1. The service should be highly available.
2. The service should be fault-tolerant.


# Design Goals (Assumption)

1. If the system fails, it will imply that all the short links will not function. Therefore, our system should be highly available.
2. URL redirection should happen in real-time with minimal latency.
3. Shortened links should not be predictable in any manner.
4. The service should be REST API accessible.
5. Analytics: How many times the URL is visited?
6. Users should be able to specify the expiration time of the URL.


# Database (Assumption)

Relational Databases or NoSQL. Service should be scalable and fault-tolerant. So, here we will go with NoSQL. A NoSQL choice would be easier to scale as they obey the distributed system and provide horizontal scaling.


# System Analysis (Assumption)

Clients- Web Browsers/Mobile app. It will communicate with the backend servers via HTTP protocol.
Load Balancer- To distribute the load evenly among the backend servers.
Web Servers- Multiple instances of web servers will be deployed for horizontal scaling.
Database- It will be used to store the mapping of long URLs to short URLs.


# Estimation of Scale and Constraints

# Rate Limits: 
We can assume that we will have 500 million new URL shortening requests per month; with a 100:1 read/write ratio, we can expect 50 billion redirections during the same period (100 * 500 million= 50 billion).

If we calculate this value for each second, i.e., queries per second (QPS) = 500 million / (30 days * 24 hours * 3600 seconds) = ~200 URLs/s
Considering the 100:1 read/write ratio, URLs redirections per second will be = 100 * 200 URLs/s = 20K/s


# Storage Estimation: 
Let us assume that we are willing to store our data (short URL + long URL) for ten years then, the number of URLs we will be storing would be = 500 million * 10 * 12 = 60 billion URLs

Now, we assume the URL's length to be 120 characters (120 bytes) on average and then add 80 more bytes to store the information about the URL. Total storage requirement = 60 billion * 200 bytes = 12TB


# length of the short URL
The length of the short URL should be at most 6 characters (of course, in the future, we can increase the length to 7, 8, or more characters, if needed. However, we will see that even with 6 characters, we can generate a very large number of unique short URLs).

# Language choice
It depends on the latency attribute. Ideally, it should be in C++ but we will go with the python as it has lots of in-built libraries to make a short_url.


# Scaling the service

# Caching
We know that our database is going to be read heavily. We have found a way to speed up the writing process, but the reads are still slow. So we have to find some way to speed up the reading process. Let’s see.

We can speed up our database reads by putting as much data in memory as possible, AKA a cache. This becomes important if we get massive load requests to a single link. If we have the redirect URL in memory, we can process those requests quickly. Our cache could store, let’s say, 20% of the most used URLs. When the cache is full, we would want to replace a URL with a more popular one.

A Least Recently Used (LRU) cache eviction system would be a good choice.
We can shard the cache too. This will help us store more data in memory because of the more machines. Deciding which thing goes to which shard can be done using “Hashing” or “Consistent Hashing.”

# Load Balancing
With multiple access systems like this one, a web server may not handle it yet. To solve this problem, I will use many web servers. Each web server will take a partial request from the user.

Since we have a large-scale system with multiple access happening simultaneously, a server may not handle the requests. To solve this problem, we can use multiple servers with a load balancer between “the client and the server” and “the server and the database” and avoid a single point of failure, and we will use multiple load balancers.


# Encoding URL
We will use (a-z, A-Z, 0–9) to encode our URLs which leads us to use base62 encoding. Now, the questions arise, what should be the length of the short URL generated to cater to 60 billion requests. Let us find out.

The longer our key, the more URLs we have, and the less we have to worry about our system ever running out. However, the point of this system is to keep our URL as short as possible. Let’s stick with 7 because even if we consumed 1000 keys a second, it would take 111 years to run out.

