# docker
optimal docker configuration for laravel environment

## Running Laravel Schedules and Queues

To run Laravel schedules and queues, follow these steps:

1. Ensure that the `supervisor` package is installed in the `php` container.
2. Add a configuration file for `supervisor` to manage Laravel schedules and queues.
3. Update the `docker-compose.yml` file to include services for running Laravel schedules and queues.

### Example `docker-compose.yml` Configuration

```yaml
version: '3.1'

volumes:
  zero-redis:
  zero-database:

networks:
  laravel:

services:

  nginx:
    networks:
      - laravel
    build:
      context: ./nginx
    container_name: zero-nginx
    ports:
      - "8088:80"
    volumes:
      - ${APP_PATH_HOST}:${APP_PATH_CONTAINER}
      - ${NGINX_CONF_FILE}:/etc/nginx/conf.d/default.conf
    depends_on:
      - php
      - postgresql

  redis:
    container_name: zero-redis
    volumes:
      - 'zero-redis:/data'
    ports:
      - "63790:6379"
    healthcheck:
      test: [ "CMD", "redis-cli", "ping" ]
    image: library/redis:6.2.1-alpine
    networks:
      - laravel

  composer:
    build:
      context: ./composer
    container_name: zero-composer
    volumes:
      - ${APP_PATH_HOST}:${APP_PATH_CONTAINER}
    working_dir: ${APP_PATH_CONTAINER}
    depends_on:
      - php
    user: laravel
    networks:
      - laravel
    entrypoint: [ 'composer' ]

  npm:
    image: node:13.7
    container_name: zero-npm
    volumes:
      - ${APP_PATH_HOST}:${APP_PATH_CONTAINER}
    working_dir: ${APP_PATH_CONTAINER}
    entrypoint: [ 'npm' ]

  php:
    networks:
      - laravel
    build:
      context: ./php
    container_name: zero-php
    volumes:
      - ${APP_PATH_HOST}:${APP_PATH_CONTAINER}
    ports:
      - "9000:9000"
    depends_on:
      - artisan

  postgresql:
    networks:
      - laravel
    container_name: zero-pgdb
    restart: unless-stopped
    tty: true
    ports:
      - "54326:5432"
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    volumes:
      - zero-database:/var/lib/postgresql/data/
    image: library/postgres:13.2-alpine
    healthcheck:
      test: [ "CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_PASSWORD}" ]
      interval: 5s
      timeout: 5s
      retries: 5

  artisan:
    build:
      context: ./php
    container_name: zero-artisan
    volumes:
      - ${APP_PATH_HOST}:${APP_PATH_CONTAINER}
    depends_on:
      - postgresql
      - schedule
      - queue
    working_dir: ${APP_PATH_CONTAINER}
    user: laravel
    entrypoint: [ 'php', '/var/www/html/artisan' ]
    networks:
      - laravel

  schedule:
    build:
      context: ./php
    container_name: zero-schedule
    volumes:
      - ${APP_PATH_HOST}:${APP_PATH_CONTAINER}
    depends_on:
      - postgresql
    working_dir: ${APP_PATH_CONTAINER}
    user: laravel
    entrypoint: [ 'php', '/var/www/html/artisan', 'schedule:run' ]
    networks:
      - laravel

  queue:
    build:
      context: ./php
    container_name: zero-queue
    volumes:
      - ${APP_PATH_HOST}:${APP_PATH_CONTAINER}
    depends_on:
      - postgresql
    working_dir: ${APP_PATH_CONTAINER}
    user: laravel
    entrypoint: [ 'php', '/var/www/html/artisan', 'queue:work' ]
    networks:
      - laravel
```

### Example `supervisord.conf` Configuration

```ini
[supervisord]
nodaemon=true

[program:schedule]
command=php /var/www/html/artisan schedule:run
autostart=true
autorestart=true
user=laravel
redirect_stderr=true
stdout_logfile=/var/www/html/storage/logs/schedule.log

[program:queue]
command=php /var/www/html/artisan queue:work
autostart=true
autorestart=true
user=laravel
redirect_stderr=true
stdout_logfile=/var/www/html/storage/logs/queue.log
```

By following these steps, you can ensure that your Laravel application runs schedules and queues efficiently within the Docker environment.

## Step-by-Step Actions to Run in a Clean Environment

1. **Clone the repository:**
   ```sh
   git clone https://github.com/uzmasterdev/docker.git
   cd docker
   ```

2. **Copy the example environment file:**
   ```sh
   cp .env.example .env
   ```

3. **Build and start the Docker containers:**
   ```sh
   docker-compose up --build -d
   ```

4. **Install PHP dependencies using Composer:**
   ```sh
   docker-compose run --rm composer install
   ```

5. **Generate the application key:**
   ```sh
   docker-compose run --rm artisan key:generate
   ```

6. **Run database migrations:**
   ```sh
   docker-compose run --rm artisan migrate
   ```

7. **Run database seeders (if any):**
   ```sh
   docker-compose run --rm artisan db:seed
   ```

8. **Install Node.js dependencies and compile assets:**
   ```sh
   docker-compose run --rm npm install
   docker-compose run --rm npm run dev
   ```

9. **Access the application:**
   Open your web browser and navigate to `http://localhost:8088`.

By following these steps, you can set up and run the Laravel application in a clean environment using Docker.
