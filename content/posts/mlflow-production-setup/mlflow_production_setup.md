Title: How to setup MLflow in production
Date: 2019-06-17
Category: training
Tags: training, machine learning, production, mlflow
Slug: mlflow-production-setup
Authors: Pedro Muñoz Botas
Summary: This guide explains how to setup a Machine Learning model into a production environment with MLflow on Ubuntu OS in about 10 minutes.
Header_Cover: images/posts/mlflow-production-setup/mlflow_logo.png
Headline: Get a Machine Learning model into production with MLflow in 10 minutes

# MLflow in production

I've run into MLflow around a week ago and, after some testing, I consider it by far the SW of the year. 
This can be very influenced by the fact that I'm currently working on the 
productivization of Machine Learning models.

Thus, I'm going to show you how to setup up MLflow in a production environment
as the one David and I have for our Machine Learning projects.

## Tracking Server Setup

The tracking server is the User Interface and metastore of MLflow. 
You can check the status of any run through this web application and 
centralize the model outputs and configurations in just one place.

It needs a database backend to store all the metadata. In our case we use 
PostgreSQL:

### Postgres

Let's create a database and a user to connect to it:

```bash
sudo apt install postgresql postgresql-contrib postgresql-server-dev-all
sudo -u postgres psql
create database mlflow;
create user ubuntu with encrypted password 'ubuntu';
grant all privileges on database mlflow to ubuntu;
```

Then, let's create the virtual environment where MLflow will be installed:

### Environment

```bash
sudo apt-get install python3-dev
sudo apt install virtualenv
mkdir ~/.venvs
virtualenv -p python3 ~/.venvs/mlflow_env
source ~/.venvs/mlflow_env/bin/activate
pip install mlflow
pip install psycopg2
```

Now we are ready to launch the tracking server

### Tracking Server

```bash
mkdir ~/mlruns
mlflow server --backend-store-uri postgresql://ubuntu:ubuntu@localhost/mlflow --default-artifact-root file:/home/mlflow/mlruns --host 0.0.0.0
```

As a backend for the web application we have use the MLflow Postgres database
that was setup before (--backend-store-uri). It stores all metadata related 
to MLflow executions, however it is also needed to define another place where
all MLmodels and configurations are stored (--default-artifact-root). As the
server IP we've set 0.0.0.0, but you can set any IP/DNS that points to the
machine where the tracking server is launched. 

You can check that everything worked fine checking your server URL in port 5000
in your browser: [http://0.0.0.0:5000](http://0.0.0.0:5000)

For the following, it is necessary to add the following line to the .bashrc
so that the subsequents runs of MLflow use this tracking server.

```bash
export MLFLOW_TRACKING_URI='http://0.0.0.0:5000'
```

Remember to activate this change with:

```bash
. ~/.bashrc
```

If everything works, let's run the tracking server as a service with supervisor:

### Supervisor

For those who don't know, supervisor is a quite interesting package to orchestrate
services in Linux OS. If you wanna know more about it, check the following url:
[Supervisor](http://supervisord.org/introduction.html)

```bash
sudo apt install supervisor
sudo mkdir -p /var/log/mlflow/tracking
```

To get the tracking server up and running, add the following lines to
/etc/supervisor/supervisord.conf (remember to replace the string "your_user"
in the following lines with your current username): 

```
[program:mlflow_tracking_server]
command = /bin/bash -c 'source "$0" && exec "$@"' /home/your_user/.venvs/mlflow_env/bin/activate mlflow server --backend-store-uri postgresql://ubuntu:ubuntu@localhost/mlflow --default-artifact-root file:/home/your_user/mlruns --host 0.0.0.0
autostart = true
autorestart = true
user = mlflow
stdout_logfile = /var/log/mlflow/tracking/stdout.log
stderr_logfile = /var/log/mlflow/tracking/stderr.log
stopsignal = KILL
```

Then restart supervisor and check status:

```bash
sudo service supervisor restart
sudo supervisorctl
```

You should get an output similar to this:

```
mlflow_tracking_server           RUNNING   pid 25517, uptime 0:00:12
```

Press Ctrl-D to exit.

## Serve a Machine Learning model in production

Once the tracking server is up and the MLFLOW_TRACKING_URI is pointing to 
it in the .bashrc, it's time to put your model into production.

Let's start creating the production environment to run the ML model:

### Environment

```bash
virtualenv -p python3 ~/.venvs/production_env
source ~/.venvs/production_env/bin/activate
pip install mlflow
pip install sklearn
```

A requirement for MLflow to work is having a Conda installation on the machine:

### Conda

```bash
wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda.sh
bash ~/miniconda.sh -b -p ~/miniconda
```

Remember to add the following line to the .bashrc to have the conda command in
your path:

```bash
export PATH=~/miniconda/bin:$PATH
```

Remember to activate this change with:

```bash
. ~/.bashrc
```

Then, let's clone an example from the official repository to show how to 
ramp up a model:

### Clone example

```bash
cd
git clone https://github.com/mlflow/mlflow
```

Now it's time to execute it:

### Run test example

```bash
cd ~/mlflow/examples/
mlflow run sklearn_elasticnet_wine -P alpha=0.3
```

This run will generate a new entry in your tracking server [http://0.0.0.0:5000](http://0.0.0.0:5000)
alongside with a new folder in which the model and the configuration is stored 
(~/mlruns/0/some_uuid). Let's check it:

```bash
ls -al ~/mlruns/0
```

Get the uuid related to the execution from the previous output and substitute
the string "your_model_id" with it in the following line:

```bash
mlflow models serve -m ~/mlruns/0/your_model_id/artifacts/model -h 0.0.0.0 -p 1234
```

What you have just done is serving your model as an HTTP endpoint in your
server IP and port 1234 (be careful not having any service listening there), so that
it is ready for receiving incoming data to return predictions. You can then query
your model with a simple curl command:

```bash
curl -X POST -H "Content-Type:application/json; format=pandas-split" --data '{"columns":["alcohol", "chlorides", "citric acid", "density", "fixed acidity", "free sulfur dioxide", "pH", "residual sugar", "sulphates", "total sulfur dioxide", "volatile acidity"],"data":[[12.8, 0.029, 0.48, 0.98, 6.2, 29, 3.33, 1.2, 0.39, 75, 0.66]]}' http://0.0.0.0:1234/invocations
```

Python using the requests module or any programming language is also fine for
getting predictions, since HTTP protocol is language agnostic.

Finally, if you want to serve it into production the only thing you need to do is adding
it to the supervisor configuration:

### Supervisor

```bash
sudo mkdir -p /var/log/mlflow/production
```

Add the following lines to /etc/supervisor/supervisord.conf (remember to replace the string "your_user"
in the following lines with your current username):

```
[program:mlflow_production]
command = /bin/bash -c 'source "$0" && exec "$@"' /home/your_user/.venvs/production_env/bin/activate mlflow models serve -m ~/mlruns/0/your_model_id/artifacts/model -h 0.0.0.0 -p 1234
autostart = true
autorestart = true
user = mlflow
stdout_logfile = /var/log/mlflow/production/stdout.log
stderr_logfile = /var/log/mlflow/production/stderr.log
stopsignal = KILL
```

Restart server and check that everything went ok:

```bash
sudo service supervisor restart
sudo supervisorctl
```

Expected output:

```
mlflow_tracking_server           RUNNING   pid 25517, uptime 0:00:12
mlflow_production                RUNNING   pid 27432, uptime 0:00:09
```
