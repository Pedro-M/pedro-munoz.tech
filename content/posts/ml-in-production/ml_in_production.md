Title: Machine Learning in Production. Where 99% of companies screw up
Date: 2019-06-04
Category: training
Tags: training, machine learning, production
Slug: ml-in-production
Authors: Pedro Muñoz Botas
Summary: People complain about Machine Learning models in production. Let's dig into what is really happening.
Header_Cover: images/posts/ml-in-production/ml_in_production.jpeg
Headline: Why people fail to deploy Machine Learning models into production

# General complaint

First of all, we'd like to thank Madrid meetups, they are unwittingly responsible of this post.
The launching of Machine Learning models into production has been the main discussion 
topic in the last meetups in Madrid. Unfortunately, it was not handled by the main
speakers but it brought the interest while networking chats instead.

More and more companies face the problem of having hundreds of models that predict lots
of things but... they are not useful neither even possible in a real "business"
environment.

# Reasons why ML projects never reach production environments

All resounding "ml to production" failures that we have seen in our work life share the
same pattern: __Wrong Born Definition__ or __WBD__ in The Gurus' jargon . Let's dig
into this concept through some great sentences:

- _"I have a lot of data"_: Do the first thing that comes to your mind with it
- _"I have a lot of money"_: Let's do some Machine Learning, but please, the most
complicated stuff that is in the market right now
- _"I have both a lot of data and a lot of money"_: We need to be a Data Driven Company,
I give you the money, please don't bother me again

Seems like not a nice scenario... This is the reality in 99% of Spanish companies.
Has anyone asked himself:

- _"What info do we need to increase our business growth?"_
- _"Is the data we have really useful for that purpose?"_
    - In case Yes: 
        - _"Is the data available in a normal business operation?"_
        - _"Do we have (or at least think about) the necessary infrastructure to handle it efficiently?"_
    - In case No: 
        - _"Have you think about gathering other data?"_
    
These are just a few examples of questions that every person in charge of making his
company grow through Data should ask himself. Appart from ourselves, I haven't seen
people asking these questions in my life. If you know someone, please comment at the
end of the post, we'd love to meet him/her. 

# Our experience bootstraping ML projects into production environments

We have successfully launched several Machine Learning projects into production and
they are still alive and providing value to our customers (forecasting of electrical
power in wind mill farms, user complaints...).

I think the reasons behind our success are more than one, but I want to remark the 
__product lifecycle definition__. If anyone wants to bootstrap a Machine Learning project
into production it is compulsory to define a strong end-to-end roadmap (data 
availability, business goal, infrastructure...). After that there is yet a lot to do,
but the most important work is done.

Just think about the questions we wrote in previous paragraphs. If you have them into
consideration, you will be closer than 99% of Spanish companies in having a successful
Machine Learning project, which provides value to your business and customers. 

# Let's bring the Know-How to the Madrid Data Scientists 

In the last meetup, a girl told us that she had to go to Barcelona to attend to the
only "ml to production" course in Spain nowadays. Those interested in that training
can access the course in the following [link](https://mlinproduction.github.io/){:target="_blank"}.
However, my question is:

__Why do we need to go to Barcelona__ (already being in Madrid) __to this kind of training?__

It seems clear that Madrid deserves a good training on these topic. We are preparing
materials for that right now. If you are interested, please follow us on Twitter
[@TheGurusTeam](https://twitter.com/TheGurusTeam){:target="_blank"} or 
[LinkedIn](https://www.linkedin.com/company/the-gurus-team){:target="_blank"}, we'll keep you
up to date of everything related to the training.

