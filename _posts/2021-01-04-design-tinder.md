---
layout: post
title:  Design Tinder
date:   2021-01-04 00:00:00 -0700
categories:
tags:
---

# Requirements

## 1. Functional Requirements

1. Account signup with phone verification.

2. A user is able to upload 5 pictures.

3. A user is provided with a list of suggestioned based on search criteria
  - distance
  - gender
  - age range
  - interests

4. like / dislike.

5. Profiles shown before should not be shown again.

6. Matched people can chat.

7. Analytics & monitoring

## 2. Non-functional Requirements

1. Highly available.

2. Highly scalable.

3. Near realtime with minimal latency.

4. Durable.

5. Consistency.

# API Specs

1. update_prfile(user_token, user_profile)
  - user_token: can be a jwt token
  - user_profile: model for profile

2. upload_pictures(user_token, img_path, img_meta)

3. update_search_preferences(user_token, search_preferences)

4. get_profiles(user_token, count)
  - get profiles based on search preferences

5. rate_profiles(user_token, profile_ratings: [])
  - profile_ratings: we can periodically call this API in batch manner so that
  to reduce the API calls.

# Design

```

[App] -> |-------------------|     |Analytics|          ------->|Push notification|
         | Routing / Gateway |                          |  |
         | Service           |---->| Chat  |-------------  |
         |                   |     |service|<--------------|-----
         |-------------------|                             |    |
                             |---->|Swipe service|----------    |
                             |                             |    |
                             |---->|Recommendation  |<------    |
                             |     |service         |<------    |
                             |                             |    |
                             |---->|Profile |---------------    |
                             |     |service |<-------------------
                             |                                  |
                             |---->|User registration |----------
                             |     |  service         |------------>|SMS gateway|

```

## 1. Gateway service

The main responsibility of the gateway service is to act as a middleman between
the app client and backend services. It also does authentication. For example, it 
reads the `user_token`, talks to the profile service to authenticate the request.

## 2. Profile service 

Stores user information & user search preferences. The table is sharded by user id.

Table schema:
- user Id (PK)
- email
- gender
- user profile image and uploaded images

How to store user images?

1. store it in the same user table as a blob column. The advantage is that there
is only 1 lookup needed to fetch the entire information. The disadvantage is that
if this is a relational database, data backup will take extraordinary amount of 
time due to the size of the data.

2. store the images separately in the file system, and store the image path in the 
user table. The advantage is reduced DB size. The DB commit log is also small, hence
write commit to the DB is fast. It is also fast to directly serve the files through
the socket to the client. The disadvantage is that the service is responsible for
doing data backup on its own. Another disadvantage is that uploading / deleting file
cannot be done in a DB transaction -> possibility of dangling file.

3. store the images into a object storage like S3, and store the image URI in the
user table. The advange is the reduced DB size, durability and scalability that 
comes for free. Also, it integrates well with CDN suppose we want to set it up 
one day. The disadvantage is dangling file due to transaction is not possible. 
Another disadvantage is increased UPL in certain cases since a client needs to 
perform 2 reads to display the image (1st read to the DB, 2nd read to the object
storage).

We can also adopt a hyprid approach. For example, for images smaller than a certain
size, we store it as a blob storage in the DB. For images that are bigger, we store
them into a object storage.

## 3. Swipe service

A bad design is to call the swipe API each time a user swipes left or right. This 
incurs unnecessary load to the service. A better design is to accumulate the swipes 
(likes and dislikes) on the app client, and call a batch API in a periodic manner
once the client has accumulated enough swipes.

Table schema:
- user_id_1 + user_id+2 : PK, the application can make sure the smaller id always precedes
    with the bigger id.
- user_id_1_liked: enum - like, dislike, unknown
- use_id_1_ts: ts of the user_id_1's action
- user_id_2_liked: enum - like, dislike, unknown
- use_id_2_ts: ts of the user_id_2's action

We design `user_id_1_liked` as a enum instead of a boolean so that we can keep 
track of both the like & dislike operations. We also keep the timestamp of when 
the user performed the action. These information could be also useful to the 
recommendation engine. For example, the recommendataion engine could resurface 
a disliked person if the dislike action has happened long time ago.

Alternatively, we can choose to only store the `liked` status and TS but not the
`disliked`. We treat `dislike` as the default action, and use a separate table
to keep a list of shown profiles and TS for a given user.

The swipe service forwards the swipe action into a stream that is later read by
the recommendation service to update the recommendation in the cache.

Once both users liked each other, the swipe service informs the chat service
that both users can now chat with each other, and also sends push notifications
to both users.

## 4. Recommendation service

The recommendation service takes user information from profile service, and stores
it in its own data store in a [geo-sharded](https://s2geometry.io/) fashion.

Hard preferences
- Distance, Gender etc

Soft preferences
- interests
- recommendation based on prior swipes
- the more active a user is, the more likely it is shown to others.

**How do we improve latency?**

One solution is to pre-calculate a set of results and populate that into a cache.
When a user requests for the cached result, it triggers the service to calculate a
new set of cached result.

Not all the previously cached result is obsolete since a user might just browse
5 out of the 100 cached results. The cache eviction policy should be based on
most recently used: the most recently displayed profile should be removed from the
cache, or be moved to the tail. In this way, new profiles are always prioritized to
be displayed to the user first.

## 5. Chat service

Protocols like XMPP enables 2 clients talk to each other efficiently. Typically
in a modern web browser setting, this is implemented as a Websocket connection to
backend services.

References:
[1](https://www.youtube.com/watch?v=XFQIW2R_Klk&feature=emb_logo)
[2](https://www.youtube.com/watch?v=nBdTBDJNOh8)
[3](https://www.youtube.com/watch?v=tndzLznxq40)
[4](https://s2geometry.io/)

