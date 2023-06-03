# infra_sprint1  

# Kittygram — социальная сеть для обмена фотографиями любимых котопитомцев.
- Приложение имеет френдли веб-интерфейс и возможность обращения по API.


## Описание
Cоциальная сеть для обмена фотографиями любимых питомцев. Это полностью рабочий проект, который состоит из бэкенд-приложения на Django и фронтенд-приложения на React.



## Технологии
Python 3.10.6
Django 3.2.3
djangorestframework 3.12.4
djoser 2.1.0 
Nginx
gunicorn


## Как развернуть проект.

- Получите доменное имя у одной из компаний-регистраторов.

- Клонируйте пакет infra_sprint1 с проектом Kittygram с GitHub на удалённый/локальный сервер.
git clone  git@github.com:sm-serega85/infra_sprint1

- в директории проекта /infra_sprint1/backend/kittygram_backend создайте файл .env
touch .env (Linux) / NUL > .env (Windows)

- -- в файле .env прописать ваш SECRET_KEY, значение специальной константы DEBUG, доменное имя сайта (см. первый пункт инструкции), IP вашего сервера в виде: 

SECRET_KEY = ваш_SECRET_KEY *(без кавычек)*

DEBUG = '' *(пустая строка - выключен режим отладки / непустая строка - включен режим отладки)*

ALLOWED_HOSTS = IP_адрес_вашего_сервера 127.0.0.1 localhost доменное_имя *(все значения указываются без кавычек)*

- Установите и активируйте виртуальное окружение в директории проекта /infra_sprint1.
python3 -m venv venv  (Linux) / python -m venv venv  (Windows) 
source venv/bin/activate

- Установите зависимости из файла requirements.txt
pip install -r requirements.txt


### backend
- В папке с файлом manage.py (/infra_sprint1/backend) c включеным виртуальным окружением

- -- соберите статику backend-приложения:
python manage.py collectstatic
 *В результате будет создана директория .../backend/static_backend/ со всей статикой бэкенд-приложения *

- -- скопируйте содержимое папки .../static_backend/ в системную директорию /var/www/infra_sprint1/, которая исользуется веб-сервером для раздачи статики *(добавьте имя пользователя)*:
sudo cp -r /home/<имя пользователя в системе>/infra_sprint1/backend/static_backend/ /var/www/infra_sprint1/

- -- выполните команду:
python3 manage.py migrate (Linux) / python manage.py migrate (Windows) 

- -- выполните команду для создания суперпользователя:
python3 manage.py createsuperuser (Linux) / python manage.py createsuperuser (Windows)

- -- запустите backend командой:
python3 manage.py runserver (Linux) / python manage.py runserver (Windows)


### Установка Node.js и пакетного менеджера npm
- Установить на сервер Node.js командами:
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\ sudo apt-get install -y nodejs 

- Проверьте, установлен ли пакетный менеджер npm командой:
npm –v 


### Установка и запуск Gunicorn
- Установите WSGI-сервер Gunicorn (при активированном виртуальном окружении)
gunicorn pip install gunicorn==20.1.0

- В директории /etc/systemd/system/ создайте файл gunicorn.service со следующим содержимым *(добавьте имя пользователя)*
sudo nano /etc/systemd/system/gunicorn.service 
````
  [Unit]

  Description=gunicorn daemon

  After=network.target

  [Service]

  User=<имя пользователя в системе>

  WorkingDirectory=/home/<имя пользователя в системе>/infra_sprint1/backend/

  ExecStart=/home/<имя пользователя в системе>/infra_sprint1/venv/bin/gunicorn --bind 0.0.0.0:8080 kittygram_backend.wsgi

  [Install]

  WantedBy=multi-user.target
````
 *Узнать путь до Gunicorn можно при активированном виртуальном окружении - which gunicorn *

- Запустите процесс gunicorn.service 
sudo systemctl start gunicorn-kittygram

- Добавьте процесс Gunicorn в список автозапуска ОС на сервере (для непрерывной работы сервера):
sudo systemctl enable gunicorn



### frontend
#### Установка и настройка Nginx
- На сервере из любой директории выполните команду:
sudo apt install nginx -y

- Запустите Nginx:
sudo systemctl start nginx 

- Установите зависимости для frontend-приложения, для этого нужно перейти в директорию /infra_sprint1/frontend и выполнить команду:
npm i

- После установки зависимостей запустите frontend командой:
npm run start

#### Файрвол ufw (идет в комплекте с операционной системой Linux / Ubuntu)
- Для определения открытых портов выполните по очереди команды:
sudo ufw allow 'Nginx Full'
sudo ufw allow OpenSSH

- Включите файрвол 
sudo ufw enable

- соберите статику frontend-приложения:

- -- В директории /infra_sprint1/frontend выполнить
npm run build 
*В результате будет создана директория .../frontend/build/ с набором файлов, который будет задействован при запросе статики браузером *

- -- Скопируйте содержимое папки .../frontend/build/ в системную директорию /var/www/, которая исользуется веб-сервером для раздачи статики *(добавьте имя пользователя)*:
sudo cp -r /home/<имя пользователя в системе>/infra_sprint1/frontend/build/. /var/www/infra_sprint1/

- Опишите настройки для работы со статикой фронтенд-приложения и проксирования запросов. В файле конфигурации веб-сервера **заменить** его содержимое следующим кодом:
sudo nano /etc/nginx/sites-enabled/default 

````
server {
    server_name IP_адрес_вашего_сервера доменное_имя; *(все значения указываются без кавычек)*


    location /api/ {
        proxy_pass http://127.0.0.1:8080;
    }


    location /admin/ {
        proxy_pass http://127.0.0.1:8080;
    }

    location /media/ {
        alias /var/www/infra_sprint1/media/;
    }


    location / {
        root   /var/www/infra_sprint1;
        index  index.html index.htm;
        try_files $uri /index.html =404;
    }

}

````
- После закрытия проверьте файл конфигурации на ошибки
sudo nginx -t 
*Данный ответ говорит, что с файлом конфигурации всё в порядке*
````
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful 
````

- Перезапустим Nginx:
sudo systemctl reload nginx 


- Установите на веб-сервер SSL-сертификат. 

**Проект должен быть доступен по IP_адрес_вашего_сервера и доменному_имени**

