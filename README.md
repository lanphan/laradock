BEETRACK LARADOCK SETUP
-----------------------

1. beetrack-web and beetrack-laradock-local are put into the same folder

   folder XYZ

       beetrack-web

       beetrack-laradock-local

       beetrack-test (optional, required if you need to run automation test)

2. run

```bash
cd beetrack-laradock-local
docker-compose up -d nginx mariadb adminer php-worker beanstalkd beanstalkd-console
```

3. exec into workspace

```bash
docker-compose exec workspace bash
```

then, run these commands below

```bash
cd beetrack-web
composer update
php artisan migrate:refresh --seed
php artisan passport:install
chown -R laradock:www-data storage
chmod -R 755 storage
php artisan key:generate 
ln -s <absolute path>/storage/app/public public/storage  
chmod -R 777 vendor/mpdf/mpdf/ttfontdata && chmod -R 755 vendor/mpdf/mpdf/graph_cache/ vendor/mpdf/mpdf/tmp/
```

+ Command #3: migrate (rollback then migrate) all, then seeding data
+ Command #4: generate oauth key for passport
+ Command #8: <absolute path> is absolute path of current directory in workspace container. This command required in order to share public files to Internet
+ Command #9: setup permission for pdf generating service

4. update /etc/hosts to add site

```bash
127.0.0.1   beetrack.local
```

5. use Chrome to open beetrack.local/ to browse

6. check database, open localhost:8080 to open Adminer, fill in:

```
   System: MySQL
   Server: mariadb
   Username: tracking
   Password: secret
   Database: tracking
```

7. Create HTTPS local environment for developers (Ubuntu)
   a. Go to XYZ/beetrack-laradock-local/nginx/ssl folder

```bash
cd nginx/ssl
```

   b. Create key (must enter passphrase on Mac)

```bash
openssl genrsa -des3 -out myssl.key 4096
``` 

   c. Create CSR:

```shell
openssl req -new -key myssl.key -out myssl.csr
```

and fill in the infomation. Common name will be your domain (beetrack.local). Note that this step will require passphrase, we will remove it in the next step (using passphrase on previous step).

   d. Remove passphrase:

```bash
cp myssl.key myssl.key.org && openssl rsa -in myssl.key.org -out myssl.key
``` 

You will be ask for passphrase one last time. Now you have 3 files in your temp folder: myssl.csr, myssl.key and myssl.key.org

   e. Create CRT:

```shell
openssl x509 -req -days 365 -in myssl.csr -signkey myssl.key -out myssl.crt
```

8. If you want to run automation tests, we're using these technologies below:

   a. Cucumber + Capybara

   b. Selenoid + Docker
    
  How to run: Start selenoid and selenoid-ui containers:

```bash
docker-compose -f docker-compose.yml -f docker-compose-test.yml up -d selenoid selenoid-ui
```

    Notes:
    + selenoid will occupy port 4444 (can change it in .env) 
    + selenoid-ui will occupy port 8088 (can change it in .env)

   c. Write tests using Cucumber and Capybara, put them all in ./beetrack-laradock-test/capybara/features

   d. Run tests:

```bash
docker-compose -f docker-compose.yml -f docker-compose-test.yml run tests
```

    Notes:
    + If test failed, Selenoid will record video for us, you can get these videos via ./results/video
    + Other result (summary result, some images file recorded by user) are stored in ./results/test_results and ./results/save_path


Notes:
=====
1. If getting error about "Permission denied", read answer of Virtlink from https://stackoverflow.com/questions/24055056/laravel-log-could-not-be-opened-failed-to-open-stream to solve

   a/ exec into workspace

   b/ run these commands in workspace container:

       b1/ go to  /var/www/beetrack-web  (or your site folder)

       b2/ sudo chown -R root:www-data storage

       b3/ sudo chmod -R ug+w storage

       b4/ (optional)

```bash
           php artisan cache:clear
           php artisan dump-autoload
           composer dump-autoload
```

2. If your environment is MacOS, you can use docker-sync in order to have better performance. Some tips:

   a. Folder XYZ shouldn't contain too much data

   b. Use ./sync.sh to run:

        + ./sync.sh install  --> install docker-sync (must have gem install already)
        + ./sync.sh up nginx mariadb adminer php-worker beanstalkd beanstalkd-console --> run docker-sync, then run docker-compose
        + ./sync.sh beetrack --> short command for whole long command above
        + ./sync.sh down  --> down docker-sync and all

3. In order to make sure owner of logging must be laradock, please switch to user laradock after exec into workspace container

```
su - laradock
```

or

```
setuser laradock bash
```


