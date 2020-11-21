# docker-lamp-stack

Code / notes written while following [How to Create PHP Development Environments with Docker Compose](https://www.youtube.com/watch?v=l0jb-N5H52A)

## Part One

### What is Docker Compose [(5:05)](https://youtu.be/l0jb-N5H52A?t=305)

- allows running of multiple containers applications
  - defined in `docker-compose.yaml` file

### Using Docker Compose [(6:42)](https://youtu.be/l0jb-N5H52A?t=402)

> `docker-compose [command]`

- Controlling the environment
  - `up` - sets up environment (-b flag to run in background)
  - `down` - tears down (removes) environment
  - `stop` - pause (temporary)
  - `start` - resume
- Monitoring and Troubleshooting
  - `ps` - overview (status) of all containers
  - `logs` - show logs (d'uh)
  - `top` - show active processes
  - `kill` - kill containers
- Executing commands on containers
  - `exec <service_name> [command]`

### Example Nginx Service [(5:45)](https://youtu.be/l0jb-N5H52A?t=345)

```yaml
version: "3.7"    # docker-compose version number
services:
  web:            # name of the service (user defined)
    image: nginx  # name of the image (find on dockerhub)
    ports:        # defines port-forwarding
      - "8080:80" # redirect all requests on host port 8080 to container port 80
```

## Part Two

### Project Requirements

#### PHP Base Requirements [(9:34)](https://youtu.be/l0jb-N5H52A?t=574)

- Web Server: Nginx
  - serve web content
- PHP-FPM
  - parse PHP for Nginx
- Database: MySQL
  - store data

#### Development Tools [(10:04)](https://youtu.be/l0jb-N5H52A?t=604)

- PHP Extensions
  - laravel and its dependencies
- Composer PHP
  - package manager
- Artisan Commands
  - create useful commands (similar to `npm run`?)

#### Contains & Images [(10:36)](https://youtu.be/l0jb-N5H52A?t=636)

- Nginx (pre-built)
- MySQL (pre-built)
- App (PHP-FPM)
  - custom Dockerfile

#### Shared Volumes [(11:48)](https://youtu.be/l0jb-N5H52A?t=708)

- Application files
  - host >> app, nginx
  - grant access to written code
- Nginx configuration file
  - host >> nginx
  - configure nginx server
- MySQL database dump
  - host >> mysql
  - database persistence

### Shared Networks [12:47](https://youtu.be/l0jb-N5H52A?t=765)

- host >> app, nginx, mysql
- one private network for all containers to communicate

### Setting Up `docker-compose.yml` [(12:52)](https://youtu.be/l0jb-N5H52A?t=772)

### App + PHP

```yaml
app:                             #service name
  build:                         # custom build from Dockerfile
    context: ./                  # relative path to build file
    dockerfile: Dockerfile       # build file name
  image: travellist              # build as image
  container_name: travellist-app # naming this container
  restart: unless-stopped        # restart policy (container always restarts)
  working_dir: /var/www/         # where application is located on container (should match nginx)
  volumes:
    - ./:/var/www/               # share current directory (host) with container
  networks:
    - travellist                 # private network name
```

### Nginx

```yaml
nginx:
  build: # custom build from Dockerfile
    context: ./ # relative path to build file
    dockerfile: Dockerfile # build file name
  image: travellist # build as image
  container_name: travellist-app # naming this container
  restart: unless-stopped # restart policy (container always restarts)
  working_dir: /var/www/ # where application is located on container (should match nginx)
  volumes:
    - ./:/var/www/ # share current directory (host) with container
  networks:
    - travellist # private network name
```

### DB

```yaml
db:
  image: mysql:5.7
  container_name: travellist-db
  restart: unless-stopped
  volumes:
      - ./docker-compose/mysql:/docker-entrypoint-initdb.d
  environment: # extracted from .env file
    MYSQL_DATABASE: ${DB_DATABASE}
    MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
    MYSQL_USER: ${DB_USERNAME}
    MYSQL_PASSWORD: ${DB_PASSWORD}
    SERVICE_TAGS: dev
    SERVICE_NAME: mysql
  networks:
    - travellist
```

### Setting Up App + PHP Container (Dockerfile)

- define 2 variables (using `ARG` directive)
- create user with same UID as host
- facilates creating files with Artisan commands (same permissions as host user)
- install required packages (system packages / dependencies for laravel)

```Dockerfile
FROM php:7.4-fpm
ARG uid=1000
ARG user=sammy
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    zip \
    unzip

RUN apt-get clean && rm -rf /var/lib/apt/lists/*

RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd

# Create system user to run Composer and Artisan Commands
RUN useradd -G www-data,root -u $uid -d /home/$user $user
RUN mkdir -p /home/$user/.composer && \
    chown -R $user:$user /home/$user

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Set working directory
WORKDIR /var/www

USER $user

```
