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

The first thing we need to configure is the environment.

### Environment

Let's create a new Conda environment as it will be the place where MLflow
will be installed:

```bash
conda create -n mlflow_env
conda activate mlflow_env
```

Then we have to install the MLflow library:

```bash
conda install python
pip install mlflow
```

Run the following command to check that the installation was successful:

```bash
mlflow --help
```

We'd like our Traking Server to have a Postgres database as a backend for
storing metadata, so the first step will be installing PostgreSQL:

```bash
sudo apt-get install postgresql postgresql-contrib postgresql-server-dev-all
```

Check installation connecting to the database:

```bash
sudo -u postgres psql
```

After the installation is successful, let's create an user and a database
for the Traking Server:

```sql
CREATE DATABASE mlflow;
CREATE USER mlflow WITH ENCRYPTED PASSWORD 'mlflow';
GRANT ALL PRIVILEGES ON DATABASE mlflow TO mlflow;
```

As we'll need to interact with Postgres from Python, it is needed to install
the psycopg2 library. However, to ensure a successful installation we need 
to install the gcc linux package before:

```bash
sudo apt install gcc
pip install psycopg2
```

The last step will be creating a directory in our local machine for our 
Tracking Server to log there the Machine Learning models and other artifacts.
Remember that the Postgres database is only used for storing metadata
regarding those models (imaging adding a model or a virtual environment
to a database). This directory is called artifact URI:

```bash
mkdir ~/mlruns
```

### Run

Everything is now setup to run the Tracking Server. Then write the following
command:

```bash
mlflow server --backend-store-uri postgresql://mlflow:mlflow@localhost/mlflow --default-artifact-root file:/home/your_user/mlruns -h 0.0.0.0 -p 8000
```

Now the Tracking server should be available a the following URL: 
[http://0.0.0.0:8000](http://0.0.0.0:8000). However, if you Ctrl-C or 
exit the terminal, the server will go down.

### Production

If you want the Tracking server to be up and running after restarts and 
be resilient to failures, it is very useful to run it as a systemd service.

You need to go into the /etc/systemd/system folder and create a new file called
mlflow-tracking.service with the following content:
 
```
[Unit]
Description=MLflow tracking server
After=network.target

[Service]
Restart=on-failure
RestartSec=30
StandardOutput=file:/path_to_your_logging_folder/stdout.log
StandardError=file:/path_to_your_logging_folder/stderr.log
ExecStart=/bin/bash -c 'PATH=/path_to_your_conda_installation/envs/mlflow_env/bin/:$PATH exec mlflow server --backend-store-uri postgresql://mlflow:mlflow@localhost/mlflow --default-artifact-root file:/home/your_user/mlruns -h 0.0.0.0 -p 8000'

[Install]
WantedBy=multi-user.target
```

After that, you need to activate and enable the service with the following
commands:

```bash
sudo mkdir -p /path_to_your_logging_folder
sudo systemctl daemon-reload
sudo systemctl enable mlflow-tracking
sudo systemctl start mlflow-tracking
```

Check that everything worked as expected with the following command:

```bash
sudo systemctl status mlflow-tracking
```

You should see an output similar to this:

```
● mlflow-tracking.service - MLflow tracking server
   Loaded: loaded (/etc/systemd/system/mlflow-tracking.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2019-09-27 09:02:11 UTC; 14s ago
 Main PID: 10357 (mlflow)
    Tasks: 10 (limit: 4915)
   CGroup: /system.slice/mlflow-tracking.service
           ├─10357 /path_to_your_conda_installation/envs/mlflow_env/bin/python /home/ubuntu/miniconda3/envs/mlflow_env/bin/mlflow s
           ├─10377 /path_to_your_conda_installation/envs/mlflow_env/bin/python /home/ubuntu/miniconda3/envs/mlflow_env/bin/gunicorn
           ├─10381 /path_to_your_conda_installation/envs/mlflow_env/bin/python /home/ubuntu/miniconda3/envs/mlflow_env/bin/gunicorn
           ├─10383 /path_to_your_conda_installation/envs/mlflow_env/bin/python /home/ubuntu/miniconda3/envs/mlflow_env/bin/gunicorn
           ├─10385 /path_to_your_conda_installation/envs/mlflow_env/bin/python /home/ubuntu/miniconda3/envs/mlflow_env/bin/gunicorn
           └─10386 /path_to_your_conda_installation/envs/mlflow_env/bin/python /home/ubuntu/miniconda3/envs/mlflow_env/bin/gunicorn

Sep 27 09:02:11 ubuntu systemd[1]: Started MLflow tracking server.
```
 
You can now restart your machine and the MLflow Tracking Server will be
up and running after this restart.

In order to start tracking everything under this Tracking Server it is
necessary to set the following environmental variable on .bashrc:

```
export MLFLOW_TRACKING_URI='http://0.0.0.0:8000'
```

Remember to activate this change with:

```
. ~/.bashrc
```

## Serve a Machine Learning model in production

Once the tracking server is up and the MLFLOW_TRACKING_URI is pointing to 
it in the .bashrc, it's time to put your model into production.

Let's start creating the production environment to run the ML model:

### Environment

```bash
conda create -n production_env
conda activate production_env
conda install python
pip install mlflow
pip install sklearn
```

Then, let's clone an example from the official repository to show how to 
ramp up a model:

### GitHub example

```bash
mlflow run git@github.com:databricks/mlflow-example.git -P alpha=0.5
```

This run will generate a new entry in your tracking server [http://0.0.0.0:8000](http://0.0.0.0:8000)
alongside with a new folder in which the model and the configuration is stored 
(~/mlruns/0/some_uuid). Let's check it:

```bash
ls -al ~/mlruns/0
```

Get the uuid related to the execution from the previous output and substitute
the string "your_model_id" with it in the following line (of course
you could do that searching for the uuid in the Tracking Server):

```bash
mlflow models serve -m ~/mlruns/0/your_model_id/artifacts/model -h 0.0.0.0 -p 8001
```

What you have just done is serving your model as an HTTP endpoint in your
server IP and port 8001 (be careful not having any service listening there), so that
it is ready for receiving incoming data to return predictions. You can then query
your model with a simple curl command:

```bash
curl -X POST -H "Content-Type:application/json; format=pandas-split" --data '{"columns":["alcohol", "chlorides", "citric acid", "density", "fixed acidity", "free sulfur dioxide", "pH", "residual sugar", "sulphates", "total sulfur dioxide", "volatile acidity"],"data":[[12.8, 0.029, 0.48, 0.98, 6.2, 29, 3.33, 1.2, 0.39, 75, 0.66]]}' http://0.0.0.0:8001/invocations
```

Python using the requests module or any programming language is also fine for
getting predictions, since HTTP protocol is language agnostic.

```python
import requests

host = '0.0.0.0'
port = '8001'

url = f'http://{host}:{port}/invocations'

headers = {
    'Content-Type': 'application/json',
}

# test_data is a Pandas dataframe with data for testing the ML model
http_data = test_data.to_json(orient='split')

r = requests.post(url=url, headers=headers, data=http_data)

print(f'Predictions: {r.text}')
```

### Production

Finally, if you want to serve it in production the only thing you need to do is adding
the systemd configuration:

```bash
[Unit]
Description=MLFlow model in production
After=network.target

[Service]
Restart=on-failure
RestartSec=30
StandardOutput=file:/path_to_your_logging_folder/stdout.log
StandardError=file:/path_to_your_logging_folder/stderr.log
Environment=MLFLOW_TRACKING_URI=http://host_ts:port_ts
Environment=MLFLOW_CONDA_HOME=/path_to_your_conda_installation
ExecStart=/bin/bash -c 'PATH=/path_to_your_conda_installation/envs/mlinproduction_env/bin/:$PATH exec mlflow models serve -m path_to_your_model -h host -p port'

[Install]
WantedBy=multi-user.target
```
