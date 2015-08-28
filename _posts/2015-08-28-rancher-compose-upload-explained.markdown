---
layout: post
title:  "Rancher Compose Builds using S3"
date:   2015-08-28 13:00:06
categories: rancher-compose
---

I'm going to provide step by step instructions to use AWS to remotely build docker containers using rancher compose 

### Requirements

Install the following software on your development machine

- docker
- rancher-compose

Create a free account with Amazon Web Services

You need to first define your application using `docker-compose.yml`

For this exercise, let us choose a simple application. I'll just use the example from docker compose [documentation](https://docs.docker.com/compose/) here.

create a directory called `composetest`. 

In that directory create a file named `docker-compose.yml` and fill in with :-




    web:
        build: .
        ports:
            - "5000:5000"
        links:
            - redis
    redis:
        image: redis




The above compose file defines a service called web, that opens up port 5000 of the container running it to the host. It also has a service link to redis, therefore the application running inside the `web` container will be able to reach the redis container by its hostname - `redis`.

Let us define the `rancher-compose.yml` file for it. This file will let you enhance your applciation definition by providing additional directives to it, like sparinkling sugar on sweets.


    web: 
        scale: 3


This file says that run this service on 3 different containers.

Next, we need to write the application itself and the steps to build it. 

Create a file named `app.py` and fill it with

        
    from flask import Flask
    from redis import Redis

    app = Flask(__name__)
    redis = Redis(host='redis', port=6379)

    @app.route('/')
    def hello():
        redis.incr('hits')
        return 'Hello World! I have been seen %s times.' % redis.get('hits')

    if __name__ == "__main__":
        app.run(host="0.0.0.0", debug=True)


This application talks to a host called `redis`, which is expected to be running a redis KV store, and increments the value of a key in the store called `hits`, and retrieves it.

This application depends on two libraries. Create a file called `requirements.txt` and fill it with

        
    flask
    redis 


Now, let us define the steps to build it. 

Create a file called `Dockerfile` and fill it with 


    FROM python:2.7
    ADD . /code
    WORKDIR /code
    RUN pip install -r requirements.txt
    CMD python app.py


These instruction define how the application container should be built

Now, start up your rancher-server 


    docker run -d -p 8080:8080 --restart=always rancher/server


This will start up rancher/server on your `localhost:8080`

NOTE: If you're running boot2docker on a mac, then run  


    open http://$(boot2docker ip):8080
    
to access the UI. Also, replace localhost with the result of 

        
    boot2docker ip


Now open a browser and go to `localhost:8080`

Open the `Infrastructure` tab, and then under the `Hosts` subtab, click on `Add Hosts`

If you're prompted to perform `Host Registration`, then assign an address that is routable from your local dev box (the $(boot2docker ip):8080 or localhost:8080 will work just fine)

Click on the `Custom` tile and then copy-paste the command provided into your terminal. This will start a rancher-agent on your dev box, effectively emulating the cloud setup.

Now, from the same `composetest` folder, run the following command to create a S3 build for your web application


    RANCHER_URL=http://localhost:8080/v1/projects/1a5/schema AWS_ACCESS_KEY_ID=$(aws-access-key) AWS_SECRET_ACCESS_KEY=$(aws-secret-key) rancher-compose up


Obviously, replace the symbols inside the dollars to actual values.

This will start the web container on the remote host(in this case, it is your development box). It first uploads the current directory to s3 (verify it by going to s3 UI, and checking for a new upload). Then, it downloads it from the remote host, where it builds the container using the files you provided. On successful build, it starts the container. Which means, if all went well, your web container must be running on your machine. Using a browser, Go to `remote_ip:5000` to verify that your app is actually running.
