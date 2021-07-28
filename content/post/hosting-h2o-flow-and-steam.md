+++
date = "2018-03-08T20:25:00+02:00"
draft = false
title = "Hosting H2O Flow and Steam"
slug = "hosting-h2o-flow-and-steam"
+++

[H2O](https://www.h2o.ai/) is an open source machine learning platform. With H2O Flow you're able to train models through a web interface, R or python. With H2O Steam you're able to deploy the trained models, allowing you to call a HTTP endpoint to predict user behavior. Are you planning to host Flow and Steam on your own server? Here are a few things we learned by doing.
<!--more-->

Getting H2O up and running is rather simple. It consists of two .jar files you have to download and run. The [docs](http://docs.h2o.ai/) are great for getting started. 

When we started, we started off by separating flow and Steam between two servers. We did this because the Steam deployments are continuously under heavy load predicting user behavior. And we don't want flow, which would be training new models using the full CPU, to interfere with these deployments.

## Steam doesn't keep deployments running
Once we had our servers up and trained our models, we started deploying the models and using them. This goes all smoothly through Steam's web interface. 

Sadly, however after a reboot of the server to increase the RAM, we found out none of the Steam deployments were actually running. When you deploy an endpoint Steam compiles it to a `.war` file, starts it and stores the process ID in its SQLite database. But Steam doesn't manage the deployment. So once the deployment is killed for whatever reason, some parts of the UI become unresponsive.

To fix this, we created a rather simple Go application called the [h2o-steam-overseer](https://github.com/ExpandOnline/h2o-steam-overseer) which is open source and can be found on github. What does it do? Using the Steam SQLite database it fetches the process ID of the deployments and checks if it is running. If it is not running it starts the deployment and updates the SQLite database with the new and correct process ID. We run the h2o-steam-overseer every 30 minutes through the crontab like so;

`30 * * * * /home/steam/h2o-steam-overseer /home/steam/steam-1.1.6-linux-amd64`


## R and Flow's saveModel and loadModel functions
The R library for H2O Flow has two functions called `saveModel` and `loadModel`. These functions are self-explanatory, they save and load a model respectively. But what isn't self-explanatory is that when you call these functions from your R studio, the models are saved and loaded on the *server* and not on the machine where R studio is running. This isn't something that's very intuitive and we make sure to store it in a directory that's recognizable eg. `~/models/customer-name/trained-model-march-2018`.


## Don't forget to backup
We simply backup the entire H2O Flow and Steam directories. If you want to be more specific;

1. For H2O Flow you'd backup wherever you saved your trained models.
2. For H2O Steam you'd backup the directory containing all model deployments which would be `~/steam/var/master/model`.
