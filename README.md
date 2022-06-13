
# IRIS-Systems-Task

Repository for IRIS Systems Team Recruitment Tasks. 

note: Some tasks can be seen under branches.

## Features

- DB files and nginx config stored in the ```data``` and ```nginx``` folders respectively.

- DB daily backups configured to be daily created at 3:00 a.m. and on launch of the db-backup service. Backups will be stored in the ```backup``` folder.

- Two-Stage Rate Limiter has been set under nginx with a burst of 12, delay of 8 requests and 5req/sec rate.

- nginx load balancer setup on IP Hash mode. 

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


## docker-compose.yml

#### Specifying docker Version ( not required )
```yml
version: "2.6"
```

#### Container configuration for the database

```yml
  services:
  db:
    image: mysql:5.7
    restart: always
    container_name: mysqldb
    volumes:
      - "./data:/var/lib/mysql:rw"
    environment:
      MYSQL_ROOT_PASSWORD: iris-task
      MYSQL_DATABASE: shop1_development
      MYSQL_USER: appuser
      MYSQL_PASSWORD: iris-task
```

| Parameter | value     | Description                |
| :-------- | :------- | :------------------------- |
| `image` | `mysql:5.7` | docker image for db |
| `retart` | `always` | so that the db stays active |
| `container_name` | `mysqldb` | name for the db, It will act as the hosts name. |
| `volumes` | `- "./data:/var/lib/mysql:rw"` | specifying a mount for persistence.|
| | | db data will be stored in data folder of the project. |
| `environment`|`MYSQL_ROOT_PASSWORD:iris-task`|password for the database.|
||`MYSQL_DATABASE: shop1_development`|Name for the default database.|
||`MYSQL_USER: appuser`|unused value :P|
||`MYSQL_PASSWORD: iris-task`|unused value :P|

#### Container configuration for the app

```yml
  appc1: &app_c
    build: ./app
    command: bundle exec rails s -p 3000 -b '0.0.0.0' && rake 
    depends_on:
      - db
    links:
      - db
    environment:
      DB_USER: root
      DB_NAME: shop1_development
      DB_PASSWORD: iris-task
      DB_HOST: db
```

| Parameter | value     | Description                |
| :-------- | :------- | :------------------------- |
| `appc1` | `&app_c` | creates a kind of reference so that the config can be reused. |
| `build` | `./app` | builds the docker file from inside the app folder |
| `depends_on` | `- db` | kind of helps in linking with the database |
| `links` | `- db` | Again, kind of helps in linking with the database |
| `volumes` | `- "./data:/var/lib/mysql:rw"` | specifying a mount for persistence.
| | | db data will be stored in data folder of the project. |
| `environment`|`DB_USER: root`|specifies the db user.|
||`DB_NAME: shop1_development`|Name of the database to use.|
||`DB_PASSWORD: iris-task`|password for the database.|
||`DB_HOST: db`|specifying the database host. :) docker automatically links it for us from the specified service(container).|

#### For cloned app containers app2 and app3
```yml
  appc2:
    <<: *app_c
  appc3:
    <<: *app_c
```
| Parameter | value     | Description                |
| :-------- | :------- | :------------------------- |
| `<<` | `*app_c` | copies the configuration from the reference specified. |

#### container configuration for the nginx
```yml
  web:
    image: nginx
    volumes:
      - "./nginx/nginx.conf:/etc/nginx/nginx.conf"
    ports:
      - "8080:8080"
    depends_on:
      - appc1
      - appc2
      - appc3
    restart: always
```
| Parameter | value     | Description                |
| :-------- | :------- | :------------------------- |
| `image` | `nginx` | docker image |
| `retart` | `always` | so that nginx reverse proxy always stays active |
| `depends_on` | `- appc1` | linking apps service as a dependency for this service |
|  | `- appc2` | |
|  | `- appc3` | |
| `volumes` | `- "./nginx/nginx.conf:/etc/nginx/nginx.conf"` | persistence for the nginx config file|

#### container configuration for the daily backup cronjob task server
```yml
  db-backup:
    image: fradelg/mysql-cron-backup
    environment:
      MYSQL_HOST: mysqldb
      MYSQL_USER: root
      MYSQL_PASS: iris-task
      MAX_BACKUPS: 15
      INIT_BACKUP: 1
      CRON_TIME: 0 3 * * *
      GZIP_LEVEL: 9
    depends_on:
      - db
    volumes:
      - "./backup:/backup"
    restart: unless-stopped
```

| Parameter | value     | Description                |
| :-------- | :------- | :------------------------- |
| `image` | `fradelg/mysql-cron-backup` | docker image for easier backup |
| `retart` | `unless-stopped` | so that the backup taking server stays active |
| `depends_on` | `db` | specified db as a dependency for this container |
| `volumes` | `- "./backup:/backup"` | specifying a mount for storing backups.|
| | | db data will be stored in data folder of the project. |
| `environment`|`MYSQL_*`| database configs.|
||`MAX_BACKUPS: 15`|Maximum number of backups to store.|
||| Server will deleted the oldest backup whenever full. |
||`INIT_BACKUP: 1` |To take a backup whenever this container is started|
||`CRON_TIME: 0 3 * * *`|Specified the time at which backup will be taken.|
|||https://crontab.guru can be used. currently set to take backups at 3:00AM daily|
||`GZIP_LEVEL: 9`|describes the compression level|


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
