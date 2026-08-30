---
url: 'https://viktorprogger.github.io/posts/xdebug-docker-config-example.md'
description: >-
  Ready to work xDebug config for local development with Docker and
  Docker-Compose
---
Once upon a time, I put together a working environment with PHP, xDebug, Docker and PhpStorm. Since then, I have been dragging it from project to project and felt happy. For those who find it difficult to set up a local environment with Docker and xDebug, I am posting this config with explanations. Here below we will create a Docker image with xDebug installed, configure PhpStorm and explain a working configuration for Docker Compose.

## Dockerfile

Let's start with the `Dockerfile`. Installing xDebug in supported php versions has become quite trivial. All you have to do is install PECL itself, call `pecl install` and then enable xDebug in php settings. For an Alpine-based image (e.g. `php:8.2-fpm-alpine`), this command looks like this:

```dockerfile
RUN apk add --no-cache linux-headers \
	&& apk add --update --no-cache --virtual .build-dependencies $PHPIZE_DEPS\
	&& pecl install xdebug \
	&& docker-php-ext-enable xdebug \
	&& pecl clear-cache \
	&& apk del .build-dependencies
```

Done, the image with xDebug has been built!

## PhpStorm

Open the settings window, go to PHP -> Servers and add a new server, be sure to specify the correct mappings.
::: note
It is a mapping between local files and files on the server. In our case, the server is the container inside which the application is located.
:::

![](/server-settings.png)

The most important parts are highlighted on the screenshot.

* **Absolute path on the server**: we should specify which path the project files in the container are available in. In the `docker-compose.yml` example above, I mounted them as a volume into the container along the path `/var/www`
* **Name**.  Yes, the name of the server in this case *plays a key role*, although this field is usually needed just for entering a human-readable name. In order to work correctly with `xDebug`, it must completely match to what we wrote in the `PHP_IDE_CONFIG` environment variable.  The config above says `serverName=docker`, so I also named the server `docker` in the PhpStorm settings. This is necessary so that PhpStorm can match local files with files on the server (in a container).

::: note
There is another option for matching them: instead of `PHP_IDE_CONFIG`, you can use matching by `Host` and `Port` fields, but then debugging scripts run from the console will not work.
:::

## docker-compose.yml

My minimal `docker-compose` config usually looks like this:

```yml
version: "3.8" 

services: 
  php: 
    build: 
      dockerfile: .docker/php/Dockerfile 
      context: ./ 
    user: "${USER_ID:-1000}:${GROUP_ID:-1000}" 
    restart: unless-stopped 
    volumes: 
      - ./:/var/www 
    working_dir: /var/www 
    environment: 
      PHP_IDE_CONFIG: ${PHP_IDE_CONFIG:-serverName=docker} 
      XDEBUG_MODE: ${XDEBUG_MODE:-off}
      XDEBUG_CONFIG: "client_host=host.docker.internal"
      XDEBUG_TRIGGER: 'yes'
    extra_hosts: 
      - host.docker.internal:${HOST_IP:-172.17.0.1} 
    dns: 
      - 1.1.1.1 
      - 8.8.8.8 
```

Setting up `xDebug` here is done via environment variables, thanks to Derik Rethans for such a convenient way. All possible xDebug settings and corresponding environment variables can be viewed on the [dedicated page](https://xdebug.org/docs/all_settings).

::: note
The `${VARIABLE:-someValue}` notation has been brought into the `docker-compose` configs from `bash`. It means the following: "return the value of the `VARIABLE` environment variable, but if it is not set or empty, return `someValue`".  In other words, `someValue` is the default value.  In order for it to work, you need to specify the characters `:-` between the name of the environment variable and this value.
:::

Changing the value of environment variables during local development is very convenient via the `.env` file. It may contain something like this:

```dotenv
# Change the name of the PHP server in the PhpStorm settings from docker to project-server
PHP_IDE_CONFIG=serverName=project-server

# Enable xDebug, because it is disabled by default
XDEBUG_MODE=develop,debug,coverage
```

The minus of this approach is that the debug-session will start with every request. In order not to start it, you have to either remove the `XDEBUG_TRIGGER` environment variable, or remove `debug` value from the `XDEBUG_MODE` environment variable. In the first case you'll still be able to start debug-sessions from your browser with get-parameters or cookies (e.g., with [the Chrome extension](https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc)).

The `extra_hosts` setting in the `docker-compose.yml` config deserves special attention.  It adds the entries specified in this setting to the `hosts` of the container. In this case it means the domain `host.docker.internal` should be searched by `172.17.0.1` IP.  The host machine is usually available exactly at this address, but there are exceptions, so the IP is also configured via an environment variable: so that it can be changed.

::: note
The `extra_hosts` setting is needed only for Linux, on Windows and MacOS, the `host.docker.internal` host is available inside the container by default
:::

## Finished project

I posted a simple project with the specified settings on github at https://github.com/viktorprogger/xdebug-sample. To check the specified settings work using it, clone it locally and follow these steps:

1. Set up PhpStorm as I [described above](#phpstorm)
2. Create a `.env` file with the following content:
   ```dotenv
   XDEBUG_MODE=debug
   ```
3. Set a breakpoint in the `index.php` file
4. Enable xDebug connection listening in PhpStorm
5. If you are using Windows or MacOS, remove `extra_hosts` from `docker-compose.yml`
6. Run the `docker-compose up -d` command.  We don't need to start services, but we need to create a network. The next command will not work without it.
7. Run the `docker-compose run --rm php php index.php` command. Before the first launch, the container image will be built, this will take a few minutes.
8. Voila, script execution is stopped at your breakpoint!

![](/xdebug-running.png)
