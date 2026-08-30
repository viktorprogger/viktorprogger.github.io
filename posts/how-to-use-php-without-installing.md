---
url: 'https://viktorprogger.github.io/posts/how-to-use-php-without-installing.md'
description: >-
  How to use different versions of PHP in development without installing it on
  you host machine. The specifics of using docker in aliases on Linux.
---
The answer is definitely simple: use Docker :) Benefits I get from using PHP like this:

* No need to bother with setting up a local environment, switching PHP versions, installing additional libraries and resolving conflicts
* One text file with command line aliases is enough for any version of PHP to work on any computer. And from here
  * Ease of transferring data between machines
  * You can safely demolish the system on your main computer, restoring work with PHP will be as simple as `git pull`

::: note
This article is about parsing the use of docker in one specific scenario, so if you are already familiar with this technology, you can safely skip it. The rest of you are welcome to the further reading.
:::

### What Aliases are

An alias is an additional or overridden command in the terminal that, when called, actually calls another command. For example, if you always write `ls -la` to list files in a folder, it makes sense to use the alias `alias ls='ls -la'`. Thus, every time you type `ls` in the terminal, `ls -la` will actually be called.\
Creating an alias is a terminal command. Those, you can write `alias ls='ls -la'` in the terminal, and it will work. But only in the current terminal session. For this to always work, you need to run such a command in all sessions. This is done differently in different shells. I will talk about how this is done in Bash, which is the default in most linux distributions. If you are using ZShell or another one, we will assume that you are aware of how aliases are created in it.

So bash. Any bash user can create a `.bashrc` file in their home directory, which is always executed when the terminal starts. Ubuntu creates it by default, and by default it makes the following entry in it:

```bash
if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi
```

That is, it includes the `.bash_aliases` file in the home directory, if it exists. I write all my aliases in `.bash_aliases` to keep them in a separate file. And now - let's start studying its contents.

### PHP

I wrote this set of aliases to work with PHP:

```bash
alias php='docker run --rm -it -u `id -u`:`id -g` --volume `pwd`:/app -w /app ghcr.io/mileschou/xdebug:8.1 php  $@'
alias php81='docker run --rm -it -u `id -u`:`id -g` --volume `pwd`:/app -w /app ghcr.io/mileschou/xdebug:8.1 php  $@'
alias php8='docker run --rm -it -u `id -u`:`id -g` --volume `pwd`:/app -w /app ghcr.io/mileschou/xdebug:8.0 php  $@'
alias php74='docker run --rm -it -u `id -u`:`id -g` --volume `pwd`:/app -w /app ghcr.io/mileschou/xdebug:7.4 php  $@'
```

I end up typing commands like `php -a` for interactive work or `php vendor/bin/phpunit` for running tests in the terminal, and everything works as if php is local on my machine. And when I need a different version of php for some project, I write, for example, `php74 vendor/bin/psalm`.

So, let's go to parse the above commands. For each version, I have written the same thing, only the version itself changes.

* `docker run` starts a new container based on the specified image (it is at the end of the command). [Docs for the `docker run`](https://docs.docker.com/engine/reference/commandline/run/). All the above options of this command are described on the same documentation page.
* `--rm` - this parameter says that the container should be removed after it finishes its work. If you do not specify this, for every `docker run` a new container will be created, after stopping it will not go anywhere, and soon you will run out of hard disk space.
* `-it` - this parameter is needed to work in an interactive command line session. If you do not specify it, then those php commands that are waiting for some kind of reaction from the user (for example, `php -a` or scripts that ask for data input) will not work.
* `-u` starts the container as a specific user. By default, the container runs as root, which can lead to permission issues with files that are mounted from the local machine and modified/created in the container. `` `id -u` `` and `` `id -g` `` returns the user and group id respectively, and in most cases it will be `1000`. These commands are enclosed in backticks so that they are executed before the alias is called. As a result, the command in the alias will be called with the following value of the user parameter: `-u 1000:1000` if your user and group id are both `1000`.
* ``--volume `pwd`:/app`` - this option mounts the current path in the file system (`` `pwd` ``) to the container at `/app` path. In the other words, your files from the current folder will be available in the container at the path `/app`.
* `-w /app` - set the path `/app` as the working folder in the container.
* `ghcr.io/mileschou/xdebug:8.1` - the name of the container image to run and its tag (version). I use this image instead of the official php because there is no XDebug in the official one, and sometimes you need debugging or calculating code coverage with tests. The tag is always specified after the colon, in this example it is `8.1`
* `php $@` is a command that runs inside the container. However, `$@` is a bash variable pointing to the line after the alias. I'll show you with an example. If I type `php vendor/bin/phpunit --foo --bar` in the terminal, the `$@` variable will be `vendor/bin/phpunit --foo --bar`.

#### PhpStorm

PhpStorm works great with containers. In order to be able to run tests, static analysis or other scripts from PhpStorm with a button, you need to add a container as an interpreter. [Docs](https://www.jetbrains.com/help/phpstorm/configuring-remote-interpreters.html#487450a9)
![phpstorm-settings-php-interpreter-docker.png](/phpstorm-settings-php-interpreter-docker.png)

However, the adventure doesn't end there. There is another common use case for php in the terminal, Composer.

### Composer

With the Composer, everything is as similar as possible. However, the command here is much longer, because environment variables and volumes are added. They are needed for the two purposes:

* to store settings and composer cache between container launches
* and to transfer data for ssh authorization from the host machine to the container.

I have two aliases for it to be able to work with a project where the first version is hardwired. Here I use official container images.

```bash
alias composer='docker run --rm -it --tty -u 1000:1000 -e COMPOSER_HOME=/composer-data/.composer -e COMPOSER_CACHE_DIR=/composer-data/.composer/cache -e SSH_AUTH_SOCK=/ssh-auth.sock --volume `pwd`:/app --volume /home/`whoami`/.composer:/composer-data/.composer --volume /run/user/1000/keyring/ssh:/ssh-auth.sock --volume /etc/passwd:/etc/passwd:ro --volume /etc/group:/etc/group:ro composer:2 $@'
alias composer1='docker run --rm -it --tty -u 1000:1000 -e COMPOSER_HOME=/composer-data/.composer -e COMPOSER_CACHE_DIR=/composer-data/.composer/cache -e SSH_AUTH_SOCK=/ssh-auth.sock --volume `pwd`:/app --volume /home/`whoami`/.composer:/composer-data/.composer --volume /run/user/1000/keyring/ssh:/ssh-auth.sock --volume /etc/passwd:/etc/passwd:ro --volume /etc/group:/etc/group:ro composer:1 $@'
```

In addition to the parameters that I have already analyzed above, several more are used here. I will not repeat myself and will focus only on the new ones.

* `-e` - this option sets the environment variable inside the container. I have several of them:
  * `COMPOSER_HOME=/composer-data/.composer` - where the Composer will look for and store its data (for example, access tokens on GitHub)
  * `COMPOSER_CACHE_DIR=/composer-data/.composer/cache` - where the Composer will look for and store its cache. This is necessary for optimization. If you do not store the cache on the host, it will re-download all packages every time. If there is a package of the required version in the cache, the Composer will simply copy the files from the cache to the project.
  * `SSH_AUTH_SOCK=/ssh-auth.sock` - path to authorization socket for ssh agent
* Plus I'm mounting a few more volumes from the host. These are pairs to the above environment variables.
  * ``--volume /home/`whoami`/.composer:/composer-data/.composer`` - save composer data on the host between container launches
  * ``--volume /run/user/`id -u`/keyring/ssh:/ssh-auth.sock`` - forward an authorization socket for the ssh agent into the container
  * `--volume /etc/passwd:/etc/passwd:ro` and `--volume /etc/group:/etc/group:ro` dump user accounts from the host in read-only mode (the `:ro` at the end is responsible for this) into the container. It is worth noting right away that this is not safe, and it is worth doing this only if you understand what you are doing and why. Personally, I believe that the official images of the composer are trustworthy, but this is just my personal opinion. And without these volumes, authorization on the GitHub via SSH with keys from the host machine will not work.

It is worth remembering that the container contains the minimum assembly required for the composer to work. This means that sometimes you will have to append the `--ignore-platform-reqs` flag to the command you are calling: `composer i --ignore-platform-reqs`.

If you have any questions or suggestions for improving aliases - [welcome to the blog chat](https://t.me/viktorprogger_channel_ru), let's chat.
