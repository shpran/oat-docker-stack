# OAT docker stack

## Images
* php:7.4.28-fmp
* nginx:1.21.6-alpine
* mysql:8.0.28
* pgsql:14.2-alpine
* redis:6.2.6-alpine
* elasticsearch:7.10.1
* opensearchproject/opensearch:1.2.4
* blackfire/blackfire:2.5.2

## Requirements
* docker
* docker-compose
* git

## Usage
1. Clone this repository to your local machine and go inside:
   ```shell
   git clone https://github.com/shpran/oat-docker-stack && cd oat-docker-stack
   ```
2. Create an `app` directory:
   ```shell
   mkdir app
   ```
   Further you can store your OAT project inside this `app` folder.
3. Make a copy of `.env.example` file and name it `.env`:
   ```shell
   cp .env.example .env
   ```
4. Configure all environment variables in `.env` file
5. Install [mkcert](https://github.com/FiloSottile/mkcert#installation)
6. Execute the following command to generate certificates:
   ```shell
   ./scripts/gencerts.sh
   ```
7. Create a network:
   ```shell
   docker network create [PROJECT_NAME]
   ```
8. Build and run docker containers:
   ```shell
   docker compose build && docker compose up -d
   ```
9. Open `hosts` file and add new host `127.0.0.1 [PROJECT_NAME].docker.loc`. For example:
   ```
   127.0.0.1 test.docker.loc
   ```
10. Now you can open `https://[PROJECT_NAME].docker.loc:[NGINX_SSL_PORT]` in your browser.

## Environment variables:
| Variable               | Example                                                          | Default |
|------------------------|------------------------------------------------------------------|---------|
| CONTAINER_PREFIX       | local                                                            | -       |
| PROJECT_NAME           | test                                                             | -       |
| TIMEZONE               | Europe/Minsk                                                     | -       |
| PATH_TO_ROOT           | /public (path relative to project folder)                        | -       |
| GITHUB_USER            | username                                                         | -       |
| GITHUB_TOKEN           | ghp_TQHc8mAcw9bgoGKzGbSAS1raSK9QSbOKJQ1Q                         | -       |
| GITHUB_EMAIL           | test@test.com                                                    | -       |
| GITHUB_NAME            | "Name Surname"                                                   | -       |
| NPM_TOKEN              | c4172d99-8a41-40d3-9e2f-54441bd1c0b0                             | -       |
| XDEBUG_CLIENT_HOST     | 192.168.100.1                                                    | -       |
| NGINX_PORT             | 8080                                                             | -       |
| NGINX_SSL_PORT         | 443                                                              | -       |
| MYSQL_ROOT_PASSWORD    | root                                                             | root    |
| MYSQL_USER             | dev                                                              | dev     |
| MYSQL_PASSWORD         | dev                                                              | dev     |
| MYSQL_DATABASE         | dev                                                              | dev     |
| MYSQL_PORT             | 3306                                                             | -       |
| POSTGRES_USER          | dev                                                              | dev     |
| POSTGRES_PASSWORD      | dev                                                              | dev     |
| POSTGRES_DB            | dev                                                              | dev     |
| POSTGRES_PORT          | 5432                                                             | -       |
| BLACKFIRE_LOG_LEVEL    | 4                                                                | -       |
| BLACKFIRE_SERVER_ID    | 65195f5c-9c97-4274-81ec-87a8ecab9bb4                             | -       |
| BLACKFIRE_SERVER_TOKEN | b2g6faycq78n2r3o48ecrrj6vq49i2ffquxoq7f520smtv8zramru8v00yk9q3hc | -       |
| BLACKFIRE_CLIENT_ID    | 2652939e-99e8-4237-a02d-a5c395d9f779                             | -       |
| BLACKFIRE_CLIENT_TOKEN | 33igg3c4dtzlb987982cikfjf796ynmhy60qsj6u1sj76l26dw5r5kclem8xdg69 | -       |

## How to access containers
Use the following command to access your containers:
```shell
docker compose exec [container] [bash|sh]
```
or this command, if you are trying to access containers outside the docker-project directory:
```shell
docker exec -it [CONTATINER_PREFIX]_[container] [bash|sh]
```

## Xdebug
The functionality only gets activated when a specific trigger is present when the request starts.  
The name of the trigger is `XDEBUG_TRIGGER`, and Xdebug checks for its presence in either `$_ENV` (environment variable),
`$_GET` or `$_POST` variable, or `$_COOKIE` (HTTP cookie name).  
Also, you can set environment variable directly on the server:
```shell
export XDEBUG_TRIGGER=1
```
Or remove it:
```shell
unset XDEBUG_TRIGGER
```

The recommended way to initiate a debugging session is by configuring your IDE to accept incoming debugging  
connections, and then use a browser extension which sets the right trigger cookie.

### Browser Extension Initiation
The extensions are:
* [Xdebug Helper for Firefox](https://addons.mozilla.org/en-GB/firefox/addon/xdebug-helper-for-firefox/) ([source](https://github.com/BrianGilbert/xdebug-helper-for-firefox)).
* [Xdebug Helper for Chrome](https://chrome.google.com/extensions/detail/eadndfjplgieldjbigjakmdgkmoaaaoc) ([source](https://github.com/mac-cain13/xdebug-helper-for-chrome)).
* [XDebugToggle for Safari](https://apps.apple.com/app/safari-xdebug-toggle/id1437227804?mt=12) ([source](https://github.com/kampfq/SafariXDebugToggle)).

Each extension adds an icon to your browser where you can select which functionality you want to trigger.  
Xdebug will continue to start debugging for every request as long as the debug toggle has been enabled.
