
# IRIS-Systems-Task

Repository for IRIS Systems Team Recruitment Tasks.

## Features

- DB files and nginx config stored in the ```data``` and ```nginx``` folders respectively.

- DB daily backups configured to be daily created at 3:00 a.m. and on launch of the db-backup service. Backups will be stored in the ```backup``` folder.

- Two-Stage Rate Limiter has been set under nginx with a burst of 12, delay of 8 requests and 5req/sec rate.

## Run Locally

Clone the project

```bash
  git clone https://github.com/Utkar5hM/iris-systems-task.git
```

Go to the project directory

```bash
  cd iris-systems-task
```

Start the docker Containers

```bash
  docker-compose up
```

migrate DB if you are launching the app for the first time using

```bash
  docker-compose run app rake db:migrate
```


## Docker Images Used:

- For database: mysql:5.7
- For reverse-proxy/load-balancer: Nginx
- For Daily DB backup using cronjob : fradelg/mysql-cron-backup
- For the rails app: ruby:2.5.1


## Screenshots

With all the 3 container runnning using IP hash method for load balancing:
![App Screenshot](https://i.imgur.com/KHqW1Go.png)

adding a example shop:
![App Screenshot](https://i.imgur.com/q2aTZF5.png)


Testing nginx rate-limiter using siege:

command:
```
siege -c 20 -r 1 -b --no-parser http://localhost:8080/

```
![App Screenshot](https://i.imgur.com/hnqXwNM.png)


Verifying Nightly Backups:
```
ls backup/ -l
```
![App Screenshot](https://i.imgur.com/jRNLYn9.png)

## Tasks Completed

- [x] Pack the rails application in a docker container image.
- [x] Launch the application in a docker container. Launch a separate container for the
database and ensure that the two containers are able to connect.
     a. The DB port should not be exposed to the host or external network. It must be
internal to the docker network only.
     b. Expose the application port to the host machine at 8080. So you should be able to
access the app at “localhost:8080”.
- [x] Launch an Nginx container, and configure it as a reverse proxy to the rails application.
Expose it at port 8080 on localhost. So now the rails application shouldn’t be accessed
directly. All requests will go through Nginx.
- [x] Now launch two more containers of the rails application. All three containers should be
able to connect to a single database container. Configure Nginx container to load balance
incoming requests between the three containers.
- [x] Enable persistence for the DB data and Nginx config files so that they are available even
when the containers go down.
- [x] Use docker-compose to easily bring these containers up together with a single command
- [x] Add requests rate limit to Nginx to limit the number of HTTP requests to the application in
a given period of time
- [x] Create a cron job to create nightly database dumps and store them on the host machine
for persistence.
