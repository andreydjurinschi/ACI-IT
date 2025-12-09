# Расширенный пайплайн в GitLab CI для Laravel

## [Цели и условие](https://elearning.usm.md/pluginfile.php/795864/mod_resource/content/1/%D0%9B%D0%B0%D0%B1%205.md)

____

1) установка Docker + развертывание GitLab

скопированная команда: 

```bash
  docker run -d \
    --hostname 192.168.100.75 \
    -p 80:80 \
    -p 443:443 \
    -p 8022:22 \
    --name gitlab \
    -e GITLAB_OMNIBUS_CONFIG="external_url='http://192.168.100.75'; gitlab_rails['gitlab_shell_ssh_port']=8022" \
    -v gitlab-data:/var/opt/gitlab \
    -v ~/gitlab-config:/etc/gitlab \
    gitlab/gitlab-ce:latest
```

сначала необходимо изменить тип сетового адаптера самой машины, чтобы gitlab был достукпен из браузера компьютера-хоста

![](https://i.imgur.com/kkrsGwj.png)

теперь мы имеем фактический лдокальный ip внутри ВМ

копируем и подставляем свой айпишник в команду выше


подождав, на хостовой машине, используя локальный ip можно перейти в графический интерфейс gitlab-a

![](https://i.imgur.com/z4AH1Ex.png)


вводим логин `root` и пароль `docker exec -it gitlab cat /etc/gitlab/initial_root_password`

![](https://i.imgur.com/UIdB2bB.png)



2) установка runner-a

```bash
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
sudo apt-get install -y gitlab-runner
```

```bash
aiai@PC:~/Desktop$ gitlab-runner -v
Version:      18.6.3
Git revision: dbac4904
Git branch:   18-6-stable
GO version:   go1.24.6 X:cacheprog
Built:        2025-11-28T21:25:01Z
OS/Arch:      linux/amd64

```

создаем раннер и регистрируем его

![](https://i.imgur.com/Tm0VXPS.png)

```bash
gitlab-runner register
```

```bash
Created missing unique system ID                    system_id=s_185e744b7715
Enter the GitLab instance URL (for example, https://gitlab.com/):
[http://192.168.100.181]: http://192.168.100.181/
Enter the registration token:
glrt-lJJDiz6sCN6WO5fpIzBkhm86MQp0OjEKdToxCw.01.120y4t8j0
Verifying runner... is valid                        correlation_id=01KBB2F8294N8QMK2D4ZFV8CAE runner=lJJDiz6sC
Enter a name for the runner. This is stored only in the local config.toml file:
[PC]: docker runner
Enter an executor: instance, docker, docker+machine, docker-autoscaler, custom, shell, ssh, parallels, virtualbox, docker-windows, kubernetes:
docker
Enter the default Docker image (for example, ruby:3.3):
php:8.2-cli
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
```

**супер класс**

![](https://i.imgur.com/ySS1NNp.png)

3) создаем проект

создав репозиторий, клонируем его локально + 'скачиваем' ларавель

```bash
aiai@PC:~/Desktop$ git clone http://192.168.100.181/root/laravel-app.git ~/laravel-app
cd ~/laravel-app
Cloning into '/home/aiai/laravel-app'...
warning: You appear to have cloned an empty repository.
aiai@PC:~/laravel-app$ git clone https://github.com/laravel/laravel ~/laravel-download
cp -r ~/laravel-download/* ~/laravel-app/
Cloning into '/home/aiai/laravel-download'...
remote: Enumerating objects: 35054, done.
remote: Counting objects: 100% (42/42), done.
remote: Compressing objects: 100% (27/27), done.
remote: Total 35054 (delta 27), reused 15 (delta 15), pack-reused 35012 (from 3)
Receiving objects: 100% (35054/35054), 10.66 MiB | 7.58 MiB/s, done.
Resolving deltas: 100% (20731/20731), done.
```
ДАЛЕЕ:

- создаем докерфайл и копируем содержимое из описания лабораторной работы

- создаем .env.testing для тестов

```bash
aiai@PC:~/laravel-app$ ls -la
total 92
drwxrwxr-x 12 aiai aiai 4096 Nov 30 21:39 .
drwxr-x--- 23 aiai aiai 4096 Nov 30 21:13 ..
drwxrwxr-x  5 aiai aiai 4096 Nov 30 21:14 app
-rwxrwxr-x  1 aiai aiai  425 Nov 30 21:14 artisan
drwxrwxr-x  3 aiai aiai 4096 Nov 30 21:14 bootstrap
-rw-rw-r--  1 aiai aiai 9270 Nov 30 21:14 CHANGELOG.md
-rw-rw-r--  1 aiai aiai 2836 Nov 30 21:14 composer.json
drwxrwxr-x  2 aiai aiai 4096 Nov 30 21:14 config
drwxrwxr-x  5 aiai aiai 4096 Nov 30 21:14 database
-rw-rw-r--  1 aiai aiai  491 Nov 30 21:15 Dockerfile
-rw-rw-r--  1 aiai aiai  921 Nov 30 21:39 .env.testing
drwxrwxr-x  7 aiai aiai 4096 Nov 30 21:13 .git
-rw-rw-r--  1 aiai aiai  414 Nov 30 21:14 package.json
-rw-rw-r--  1 aiai aiai 1284 Nov 30 21:14 phpunit.xml
drwxrwxr-x  2 aiai aiai 4096 Nov 30 21:14 public
-rw-rw-r--  1 aiai aiai 3911 Nov 30 21:14 README.md
drwxrwxr-x  5 aiai aiai 4096 Nov 30 21:14 resources
drwxrwxr-x  2 aiai aiai 4096 Nov 30 21:14 routes
drwxrwxr-x  5 aiai aiai 4096 Nov 30 21:14 storage
drwxrwxr-x  4 aiai aiai 4096 Nov 30 21:14 tests
-rw-rw-r--  1 aiai aiai  436 Nov 30 21:14 vite.config.js
```

```bash
aiai@PC:~/laravel-app/tests/Unit$ cat ExampleTest.php 
<?php

namespace Tests\Unit;

use PHPUnit\Framework\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic test example.
     */
    public function test_that_true_is_true(): void
    {
        $this->assertTrue(true);
    }
}

```

тесты на месте

```bash
aiai@PC:~/laravel-app$ cat .gitlab-ci.yml 
stages:
  - test
  - build
services:
  - mysql:8.0
variables:
  MYSQL_DATABASE: laravel_test
  MYSQL_ROOT_PASSWORD: root
  DB_HOST: mysql
test:
  stage: test
  image: php:8.2-cli
  before_script:
    - apt-get update -yqq
    - apt-get install -yqq libpng-dev libonig-dev libxml2-dev libzip-dev unzip git
    - docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath
    - curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
    - composer install --no-scripts --no-interaction
    - cp .env.testing .env
    - php artisan key:generate
    - php artisan migrate --seed
    - cp .env .env.testing
    - php artisan config:clear
  script:
    - vendor/bin/phpunit
  after_script:
    - rm -f .env

```

этот файл .gitlab-ci.yml — конфигурация для GitLab CI/CD пайплайна, который автоматизирует тестирование и сборку Laravel приложения.

Структура файла:

*stages* — этапы выполнения:

*test* — запуск тестов
*build* — сборка приложения
*services* — дополнительные сервисы:

*mysql:8.0 *— база данных MySQL для тестов
*variables* — переменные окружения:

*MYSQL_DATABASE: laravel_test* — имя тестовой БД
*MYSQL_ROOT_PASSWORD: root* — пароль root
*DB_HOST: mysql* — хост для подключения
*test job* — основная задача тестирования:

*before_script* — подготовка окружения:

Обновление пакетов
Установка зависимостей (libpng, libonig, libxml2, libzip, git)
Установка PHP расширений для работы с БД и криптографией
Установка Composer и зависимостей проекта
Создание .env из .env.testing
Генерация ключа приложения
Миграция БД и заполнение тестовых данных
script — выполнение тестов:

*vendor/bin/phpunit* — запуск PHPUnit тестов
after_script — очистка после выполнения:

Удаление файла .env

пайплайн автоматически тестирует Laravel приложение при каждом коммите в репозиторий

- добавляем - коммитим - пушим

```bash
aiai@PC:~/laravel-app$ git push -u origin main
Username for 'http://192.168.100.181': root
Password for 'http://root@192.168.100.181': 
Enumerating objects: 75, done.
Counting objects: 100% (75/75), done.
Delta compression using up to 4 threads
Compressing objects: 100% (58/58), done.
Writing objects: 100% (75/75), 42.67 KiB | 3.28 MiB/s, done.
Total 75 (delta 1), reused 0 (delta 0), pack-reused 0
To http://192.168.100.181/root/laravel-app.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```




![](https://i.imgur.com/lREcwAM.png)

```bash
$ cp .env.testing .env
$ php artisan key:generate
   INFO  Application key set successfully.  
$ php artisan migrate --seed
   INFO  Preparing database.  
  Creating migration table ..................................... 584.07ms DONE
   INFO  Running migrations.  
  0001_01_01_000000_create_users_table ......................... 975.04ms DONE
  0001_01_01_000001_create_cache_table ......................... 279.77ms DONE
  0001_01_01_000002_create_jobs_table .......................... 811.54ms DONE
   INFO  Seeding database.  
$ cp .env .env.testing
$ php artisan config:clear
   INFO  Configuration cache cleared successfully.  
$ vendor/bin/phpunit
PHPUnit 11.5.44 by Sebastian Bergmann and contributors.
Runtime:       PHP 8.2.29
Configuration: /builds/root/laravel-app/phpunit.xml
..                                                                  2 / 2 (100%)
Time: 00:00.259, Memory: 28.00 MB
OK (2 tests, 2 assertions)
Running after_script
00:02
Running after script...
$ rm -f .env
Cleaning up project directory and file based variables
00:02
Job succeeded
```
