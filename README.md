# iris-systems-task


## To start:

run
```
docker-compose up
```

migrate db for the first time you launch using
```
docker-compose run app rake db:migrate
```

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

