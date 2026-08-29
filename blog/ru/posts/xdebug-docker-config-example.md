---
url: 'https://viktorprogger.github.io/blog/ru/posts/xdebug-docker-config-example.md'
description: >-
  Готовый работающий конфиг xDebug для локальной разработки с Docker и
  Docker-Compose
---
Когда-то давно я собрал рабочий комбайн из PHP, xDebug, Docker и PhpStorm. С тех пор я таскаю его из проекта в проект и горя не знаю. Для тех, у кого настройка локального окружения с докером и xDebug вызывает сложности, выкладываю этот конфиг с пояснениями. Ниже мы напишем образ контейнера с установленным в нем xDebug, настроим PhpStorm и разберем рабочую конфигурацию Docker Compose.

## Dockerfile

Начнем с Dockerfile. Установка xDebug в поддерживаемых версиях php стала совсем тривиальной. Надо лишь установить PECL и вызвать `pecl install`, после чего - включить xDebug в настройках php. Для образа, основного на Alpine (например, `php:8.2-fpm-alpine`), эта команда выглядит вот так:

```dockerfile
RUN apk add --no-cache linux-headers  \
    && apk add --update --no-cache --virtual .build-dependencies $PHPIZE_DEPS \  
    && pecl install xdebug \  
    && docker-php-ext-enable xdebug \  
    && pecl clear-cache \  
    && apk del .build-dependencies
```

Готово, образ с xDebug собран!

## PhpStorm

Открываем окно настроек, идем в PHP -> Servers и добавляем новый сервер, обязательно указывая корректные маппинги.
:::note
Маппинг - это сопоставление локальных файлов и файлов на сервере. В нашем случае сервер - это контейнер, внутри которого находится приложение.
:::

![](/blog/server-settings.png)

Самые важные части на скриншоте выделены.

* **Absolute path on the server**: указываем, по какому пути доступны файлы проекта в контейнере В примере `docker-compose.yml` выше я прокинул их томом в контейнер по пути `/var/www`
* **Name**. Да, название сервера в данном случае играет *ключевую* роль, хотя обычно это поле - исключительно для ввода человекопонятного названия. Для корректной работы с `xDebug` оно должно полностью совпадать с тем, что мы написали в переменную окружения `PHP_IDE_CONFIG`. В конфиге выше написано `serverName=docker`, поэтому сервер в настройках PhpStorm я назвал тоже `docker`. Нужно это для того, чтобы PhpStorm смог сопоставить локальные файлы с файлами на сервере (в контейнере).

  :::note
  Есть и другой вариант их сопоставления: вместо `PHP_IDE_CONFIG` можно использовать сопоставление по полям `Host` и `Port`, но тогда не будет работать отладка скриптов, запускаемых из консоли.
  :::

## docker-compose.yml

Минимальный конфиг `docker-compose` у меня выглядит обычно вот так:

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

Настройка `xDebug` здесь происходит через переменные окружения, спасибо Дерику Ретансу за столь удобный способ. Все возможные настройки xDebug и соответствующие переменные окружения можно посмотреть на [специальной странице](https://xdebug.org/docs/all_settings).

:::note
Нотация `${VARIABLE:-someValue}` принесена в конфиги `docker-compose` из `bash`. Значит она следующее: "вернуть значение переменной окружения `VARIABLE`, но если она не установлена или пуста, вернуть `someValue`". Другими словами, `someValue` - значение по умолчанию. Чтобы оно сработало, нужно указать символы `:-` между названием переменной окружения и этим значением.
:::

Изменять значение переменных окружения при локальной разработке очень удобно через файл `.env`. Он может содержать что-то вроде такого:

```dotenv
# Изменим название сервера PHP в настройках PhpStorm с docker на project-server
PHP_IDE_CONFIG=serverName=project-server

# Включим xDebug, ведь по умолчанию он выключен
XDEBUG_MODE=develop,debug,coverage
```

Минус этого подхода: чтобы debug-сессия не запускалась, надо либо убрать переменную окружения `XDEBUG_TRIGGER`, либо убрать `debug` из `XDEBUG_MODE`. В первом случае вы оставите себе возможность запускать дебаг-сессию через get-параметры и cookie, например с помощью [расширения для Chrome](https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc).

Отдельного внимания заслуживает настройка `extra_hosts` в конфиге `docker-compose.yml`. Она добавляет в `hosts`  контейнера указанные в ней записи, в данном случае - что домен `host.docker.internal` следует искать по IP `172.17.0.1`. Обычно хост-машина доступна именно по этому адресу, но бывают и исключения, поэтому конфигурируется IP тоже через переменную окружения: чтобы ее можно было изменить.

:::note
Настройка `extra_hosts` нужна только для Linux, на Windows и MacOS внутри контейнера по умолчанию доступен хост `host.docker.internal`
:::

## Готовый проект

Простой проект с указанными настройками я выложил на гитхаб по адресу https://github.com/viktorprogger/xdebug-sample. Чтобы проверить на нем, что указанные настройки работают, склонируйте его к себе и выполните следующие шаги:

1. Настроить PhpStorm как я [описывал выше](#phpstorm)
2. Создать файл `.env` со следующим содержимым:
   ```
   XDEBUG_MODE=debug
   XDEBUG_TRIGGER=yes
   ```
3. Поставить брейкпоинт в файле `index.php`
4. Включить прослушивание коннектов от xDebug в PhpStorm
5. Удалить `extra_hosts` из `docker-compose.yml,`если вы используете Windows или MacOS
6. Запустить команду `docker-compose up -d`. Поднимать сервисы нам ни к чему, а вот сеть создать надо.
7. Запустить команду `docker-compose run --rm php php index.php`. Перед первым запуском соберется образ контейнера, это займет несколько минут.
8. Вуаля, выполнение скрипта остановилось на вашем брейкпоинте!

![](/blog/xdebug-running.png)
