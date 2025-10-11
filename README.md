# Dyutas Project

## About the git submodule system that we had talked about

Not gonna use submodule because of known issues about it.
I don't know the exact reason but people say it makes the project more complex to manage.

## 1. Development Environment

### 1.1. Recommended local repository structure

Gonna use relative path in docker-compose file,
so the directory structure is recommended to be like this one below.

```
dyutas_project
├── dyutas_auth
├── dyutas_game
├── shakuni-dyutas-fe
├── dyutas-ai
├── dyutas-stat
└── dyutas-socket
```

### 1.2. docker-compose

With docker-compose file in this repository's root directory,
you can run whole Dyutas system.

First clone each Dyutas service repository.
Then install docker, and compose up.

```bash
    docker compose pull

    docker compose build

    docker compose up -d # which -d means it's gonna run in background.
```

If you wanna develop your service in your local machine,
Just comment out your service in docker-compose file
and adjust nginx.conf in dev_env directory to forward the request to your local host network
rather than docker network.

### 1.3. .env file example

```
AUTH_RDB_PASSWORD=dyutas_auth_pg_001
AUTH_DOCKERFILE_PATH=../dyutas_auth
```

### 1.4. nginx

Thanks to Docker, it was very easy to configure Nginx as a reverse proxy web server
along with SSL features.

Just run the nginx container and access to nginx from your local browser.
It just forwards every request to each service and does the SSL termination job.

nginx config file basically looks easy to do basic proxy server things.
Only the part below is needed to be adjusted when you wanna forward the request to your local host.
Just be careful about the port number.

```
...
    location /auth {
        proxy_pass http://auth-app:8020; # forwarding to docker network
        # proxy_pass http://host.docker.internal:8020; # forwading to local host(macos version)
...
```

Just go to 'https://localhost:8010/(your service url)/(blah blah)' and start developing!

### 1.5. local dev network config

For MacOS, go to '/etc/hosts' file and add below ones.

```
127.0.0.1 local.dyutas.com
127.0.0.1 local-api.dyutas.com
```

#### API servers use separate subdomain

| service type | origin                            |
| ------------ | --------------------------------- |
| default(FE)  | https://local.dyutas.com:8010     |
| API          | https://local-api.dyutas.com:8010 |

#### Port and base URL

| service  | host network port | docker network port | url     |
| -------- | ----------------- | ------------------- | ------- |
| web      | 8010              | 443                 |         |
| auth-app | X                 | 8020                | /auth   |
| auth-rdb | 8025(for admin)   | 8025                |         |
| game-app | X                 | 8030                | /game   |
| game-rdb | 8035(for admin)   | 8035                |         |
| ai-app   | X                 | 8040                | /ai     |
| stat-app | X                 | 8050                | /stat   |
| socket   | X                 | 8060                | /socket |
| fe       | X                 | 8070                |         |
