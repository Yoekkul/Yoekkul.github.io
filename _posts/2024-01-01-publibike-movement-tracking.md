---
layout: post
title:  "Mapping movement of PubliBike bike-sharing in zurich"
date:   2024-01-01 12:01:40 +0100
categories: web
---

### F.A.Q.
* **Can the bike position be tracked during a ride?**
No! The track the bike follows is only an estimate. Only the start point, end point, start time and end time can be publicly accessed. Using this information I find the shortest path between the two PubliBike stations and use that to visualize the taken path.
* **Where was the data obtained?**
PubliBike offers a publicly visible API, if you are curious you can check it out here! [PubliBike API](https://api.publibike.ch/v1/static/api.html).
For this project I call this API once every minute, and save the results for further processing.
* **Where can I find the code for this project?**
The code can be found in my personal [GitHub repository](https://github.com/Yoekkul/publibiker)