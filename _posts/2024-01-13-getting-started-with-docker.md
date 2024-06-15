---
layout: post
title: Getting Started - Build and deploy a ridiculously simple python web application with Docker
categories: [Tutorial,Devops, Cloud, SRE]
author: tgangte
---

Let's build a simple and straightforward website that's written in Python. It's a real site, quite basic, and greets visitors with a 'Hello’ greeting with some small talk of the weather. Docker is perfect for running and containerizing this application, so that we can deploy it anywhere later. 

Here are the objectives of this article: 
* Learn to create a simple python web application
* Learn to create a docker image 
* Learnt to deploy and distrubite a docker image

## Lets build the python web application
Open your favorite text editor (VIM on Macbook is used here) and paste the application code, and save it as webapp.py in a clean directory. I used a directory called python-docker-homepage. This will become your docker image name later.  
```python
from flask import Flask
app = Flask(__name__)
@app.route('/')
def home():
    return '<h1>Hello and welcome to your new homepage! The weather is lovely isnt it!</h1>'
if __name__ == "__main__":
    app.run(debug=False)
```
We are using the python Flask web framework here, it listens on / endpoint and returns with the Hello message.  
You also need a requirements.txt file that will contain all the modules we will need. 
```
flask==3.0.1
gunicorn==21.2.0
```
Your folder should now have two files, webapp.py and requirements.txt
```
> ls
requirements.txt	webapp.py
```
Now lets run the python webapp the traditional way
```bash
>pip  install -r requirements.txt         #this will install the dependencies  

>gunicorn --bind 0.0.0.0:8000 webapp:app  #this will run the webapp
[2024-06-11 16:04:16 -0700] [14688] [INFO] Starting gunicorn 20.1.0
[2024-06-11 16:04:16 -0700] [14688] [INFO] Listening at: http://0.0.0.0:8000 (14688)
[2024-06-11 16:04:16 -0700] [14688] [INFO] Using worker: sync
[2024-06-11 16:04:16 -0700] [14689] [INFO] Booting worker with pid: 14689
```
 Now if you open http://127.0.0.1:8000 on your web browser, you will see the application running. 
 So our web application runs fine, but it is not production ready!  This is only a local development server. 

## Lets dockerize this web application 


By dockerize, I mean we will get this application cloud ready! We will package it in a "docker image" which can then be deployed seamlessly in Kubernertes or any other cloud native provider. 
Download and install docker desktop, follow the official guides for your operating system. https://docs.docker.com/get-docker/

This section is quite simple in that that you only need two commands, "docker init" and  "docker compose up --build" to build your image and run your container. Docker init works only with Docker Desktop as of writing this, so that's what you need. 
Run "docker init" in the same directory where your webapp.py and requirements.txt reside. Follow the prompts that are asked by the docker init command, most of the default values should be good, so press enter on each step. 

```bash
> docker init
Welcome to the Docker Init CLI!
This utility will walk you through creating the following files with sensible defaults for your project:
  - .dockerignore
  - Dockerfile
  - compose.yaml
  - README.Docker.md
Let's get started!
? What application platform does your project use? Python
? What version of Python do you want to use? 3.9.6
? What port do you want your app to listen on? 8000
? What is the command you use to run your app? gunicorn 'webapp:app' --bind=0.0.0.0:8000
✔ Created → .dockerignore
✔ Created → Dockerfile
✔ Created → compose.yaml
✔ Created → README.Docker.md
→ Your Docker files are ready!
  Review your Docker files and tailor them to your application.
  Consult README.Docker.md for information about using the generated files.
What's next?
  Start your application by running → docker compose up --build
  Your application will be available at http://localhost:8000

```
This init command creates a few files for you, view the Dockerfile and compose.yaml to understand better. 

Now lets run the next command to actually run the docker container. This will run the application "detached", in the background. 

```bash
> docker compose up --build -d

[+] Building 2.2s (12/12) FINISHED                                                                                                                            docker:desktop-linux
 => [server internal] load build definition from Dockerfile                                                                                                                   0.0s
 => => transferring dockerfile: 1.68kB                                                                                                                                        0.0s
 => [server] resolve image config for docker-image://docker.io/docker/dockerfile:1                                                                                            1.4s
 => CACHED [server] docker-image://docker.io/docker/dockerfile:1@sha256:a57df69d0ea827fb726649162813635de6f17269be781f696fbfdf2d83dda33e                                      0.0s
 => [server internal] load metadata for docker.io/library/python:3.9.6-slim                                                                                                   0.6s
 => [server internal] load .dockerignore                                                                                                                                      0.0s
 => => transferring context: 671B                                                                                                                                             0.0s
 => [server base 1/5] FROM docker.io/library/python:3.9.6-slim@sha256:4115592fd52679fb3d9e8c513cae33ad3fdd64747b64d32b504419d7118bcd7c                                        0.0s
 => [server internal] load build context                                                                                                                                      0.0s
 => => transferring context: 139B                                                                                                                                             0.0s
 => CACHED [server base 2/5] WORKDIR /app                                                                                                                                     0.0s
 => CACHED [server base 3/5] RUN adduser     --disabled-password     --gecos ""     --home "/nonexistent"     --shell "/sbin/nologin"     --no-create-home     --uid "10001"  0.0s
 => CACHED [server base 4/5] RUN --mount=type=cache,target=/root/.cache/pip     --mount=type=bind,source=requirements.txt,target=requirements.txt     python -m pip install   0.0s
 => CACHED [server base 5/5] COPY . .                                                                                                                                         0.0s
 => [server] exporting to image                                                                                                                                               0.0s
 => => exporting layers                                                                                                                                                       0.0s
 => => writing image sha256:80a1676ca3741b72d0053473bf0dfae610ff1a67db15d5d195e5bef46dd0620b                                                                                  0.0s
 => => naming to docker.io/library/python-docker-homepage-server                                                                                                              0.0s
[+] Running 1/1
 ✔ Container python-docker-homepage-server-1  Started   

```
Now open the following links localhost:8000 or  http://127.0.0.1:8000 on your web browser and you will be greeted by your web application. 
![](/images/hello-and-welcome-blog001.png)


Here are a couple of commands to manage your application. 
```bash
>docker ps   #see your docker images 
CONTAINER ID   IMAGE                           COMMAND                  CREATED          STATUS          PORTS      NAMES
39b445a106d4   python-docker-homepage-server   "/bin/sh -c 'gunicor…"   58 minutes ago   Up 2 minutes    0.0.0.0:8000->8000/tcp   python-docker-homepage-server-1
>docker compose down   # shut down your application 
```
## Lets publish our image to to a docker registry

Now let us publish this image to the docker registry, so that we can easily distibute  our software and deploy it in kubernetes later! 
```bash
>docker login #create a free docker account on hub.docker.com 
>docker image ls  #find out your docker image name 
REPOSITORY                      TAG       IMAGE ID       CREATED             SIZE
python-docker-homepage-server   latest    300424b1b196   4 minutes ago       124MB 
>docker tag python-docker-homepage-server eternallycurious/python-docker-homepage-server #docker tag imagename YOUR-USER-NAME/image-name
>docker push eternallycurious/python-docker-homepage-server   #this pushes to the repository                            
Using default tag: latest
The push refers to repository [docker.io/eternallycurious/python-docker-homepage-server]
9ad9eb2501f9: Pushed 
86c485a03b6d: Mounted from library/python 
latest: digest: sha256:b90048b139527619d53559c8a75b9g1323481fb54886d8142b8a3956754eb408 size: 2202
```
This completes the publication to a registry. Effectively, "eternallycurious/python-docker-homepage-server" becomes a unique string that can be used to download the application. It is also searchable on the docker hub website. https://hub.docker.com/r/eternallycurious/python-docker-homepage-server 

Now this command can be run in any machine in the world, and the application will be deployed and run there! Run this to stop the container: "docker stop CONTAINERID"

```bash
>docker run -dp 0.0.0.0:8000:8000 eternallycurious/python-docker-homepage-server
```




