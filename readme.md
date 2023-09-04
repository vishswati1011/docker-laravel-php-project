Lets start Laravel PHP Project

1. First create empty folder for project

2. create docker-compose file in which mention the container we need in services
 
 version: "3.9"

3. services:
    server:
    php:
    mysql:
    composer:


4. here we need  a server for this we will use  nginx server and we will use docker nginx image
nginx server configuration in docker-compose file

server:
    image: 'nginx:stable-alpine'
    ports:
      - '8000:80'
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro

5. also add nginx.conf file

Now lets create php custome image becuase docker hum it not available

6. create php.dockerfile to dockerfile folder and bind it with docker-compose file

php:
    build: 
      context: ./dockerfiles
      dockerfile: php.dockerfile
    volumes:
      - ./src/:var/wwww/html:delegated

    <!-- ports:
      - '3000:9000'   -->

Why we are using container port 9000 because the php veriosn which we are using is run on 9000 port same as nginx run on 80 port 
But we will not use port on php conatiner because nginx is connect with container to container


7. Mysql container setup
   create mysql.env in env folder
  
   mysql:
    image: mysql:5.7
    env_file:
      - ./env/mysql.env


8. composer container setup
   will create dockerfile

    composer:
    build:
      context: ./dockerfiles
      dockerfile: composer.dockerfile
    volumes:
      - ./src/var/www/html  

9. now run the composer container to create the laravel project

$ docker-compose run --rm composer create-project --prefer-dist laravel/laravel .

the above command will create laraval project inside composer container and we are using volume in composer container which will copy all the code of laravel project from composer container which is inside /var/www/html  to our local src folder 


10. now go inside src folder and open .env file and change the username and password same as we used in mysql.env file

DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret

11.  now we run docker-compose up
  but our nginx dont know about the src code for the we add another volume in server container
  volumes:
      - ./src/:var/www/html
  this will bing our source code to root /var/www/html/public     
  and /var/www/html/public  is configure in nginx file

  but i dont want to run all the server so we use 
  $ docker-compose up -d server php mysql

  but instead of runnig all thre server we can add depends_on feature on nginx server and then run only one server

  $ docker-compose up -d server 

  one more thing if you want on run with update changes you you can use --build with run command

  $ docker-compose up -d --build server 
  $ docker-compose down
  $ docker-compose up -d --build server
  $ docker-compose run --rm artisan migrate

