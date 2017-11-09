---
layout: post
comments: true
title:  "A New Automated Database"
excerpt: "A Description of Oracle 18c"
date:   2017-11-08 11:00:00
mathjax: false
---

Larry Ellison, the chief technology officer of Oracle, announced an autonomous Oracle cloud database. It was claimed that the new database would deliver “unprecedented security and performance —at a significantly low cost” [1]. This article will take a look at how these promises could be fulfilled through automation.

## Automatic Patching

The autonomous database is able to patch itself automatically so that no human intervention is needed [2]. Human beings are prone to make mistakes. For instance, in May, a failure to patch software promptly resulted in the notorious Equifax breach. The programmer maintaining the Equifax site might be either inherently careless or underpaid. On the contrary, such pilot error would not happen in an autonomous database because, as Larry Ellison put, “there’s no pilot at all” [3]. And labor costs can be saved since few system maintainers are needed. This is a good news for companies that are not willing to hire a responsible, and meanwhile expensive, system maintainer.

What makes the autonomous database truly automatic is that the self-patching is done online. A system does not need to go off-line to get an update [2]. It is annoying to arrange a downtime system update [3]. For some businesses, customers in different time zones keep visiting their websites, and often times, a company would need to wait for weeks, even months to have a chance to let their computers go off-line. During that period of time, the system is very vulnerable to cyber-attacks for the simple reason that the specific vulnerability would be widely known after the patch is released. Technically, it is easy to exploit a known flaw to attack a unpatched system.

## Automate Performance Tuning

Performance tuning is routine for database administrators. For example, if queries of one pattern are made by users every day, in order to reduce the overall response time, database administrators will have to carefully review these queries, find out the pattern, and do something to improve the database performance. For instance, they could build new views and tables, or modify the structure of a database [4].  It takes experienced database administrators to effectively improve database performance. 
For some tasks, high-quality machine learning algorithms could do as well as human beings. Think about image recognition, natural language processing, and automated driving technology. Database performance tuning is another task that machine learning could do something comparable to human beings. In the light of this statement, the Oracle autonomous database makes use of machine learning algorithm to do performance tuning by itself [3].  It could collect data from event logs, use machine learning algorithms to analyze this data and, afterward, immediately implement optimizations to itself. 

The autonomous database makes it easier to implement machine learning techniques. In an autonomous database, event logs are generated and analyzed within an identical system. As a result, no data prepossession is needed before data analyzing. In fact, data prepossession is both tedious and time consuming for machine learning scientists. 

Besides being free of data prepossession, an autonomous database has another advantage over a manual machine learning approach: it could immediately use analysis results to optimize a database. While in a manual practice, even if machine learning researchers are able to give out good optimization suggestions, it still takes more time and effort to implement these changes to a database. The situation is worse if those researchers are not familiar with a database system because they will have to turn to someone else.

## Anomaly Detection

Similar to automatic performance tuning, the new Oracle database could also do automatic anomaly detection using its underlying machine learning techniques. By auditing events, it would be able to identify suspicious behaviors and immediately react to them. For example, if someone in Ukraine remotely logged into the database of an American company and tried to delete a number of tables, the autonomous database would request a stronger authentication from that person and back up these tables in the meantime [3]. Such a fast defensive reaction is impossible without an automatic system. Human beings might be able to notice an anomaly, but it is difficult for a person to respond promptly.  An attack might already have been going on for days when detected by a database administrator.

## Summary

Oracle has depicted a promising technology. Computers outperform human beings in their accuracy and low-latency. It is an interesting idea to build an automatic system to eliminate security flaws caused by human error. Although it is not yet clear that how much the Oracle autonomous database could outperform regular ones, I believe well-implemented automation will be able to add an additional fence to keep security threats away.
The autonomous Oracle database is only available on the Oracle public cloud or a private enterprise cloud offered by Oracle. You cannot install it on your laptop [5]. Yet, the price of the Oracle cloud service should be reasonable, as Larry Ellison repeatedly stressed the low cost of their products compared to similar services offered by Amazon [3]. For companies that could not afford a database security expert, by using a lower-cost autonomous database, at least they could improve the baseline security level greatly. 

## Reference

[1] Oracle (n.d.). The World’s #1 Database Is Now the World’s First Self-Driving Database. Retrieved from http://www.oracle.com

[2] Peter, B. (2017, Oct 1) Oracle announces a new automated database that can patch cybersecurity flaws itself. Business Insider. Retrieved from http://www.businessinsider.com

[3] Ellison, L (2017, Oct 3) Larry Ellison Oracle OpenWorld 2017 Keynote. Retrieved from https://www.oracle.com/openworld/index.html

[4] Database administrator (n.d.). Wikipedia. Retrieved Oct 26, 2017, from https://en.wikipedia.org/wiki

[5] Hill, R. (2017, Oct 8) Oracle’s automated database is a minimum viable release – analyst. The Register. Retrieved from http://www.theregister.co.uk
