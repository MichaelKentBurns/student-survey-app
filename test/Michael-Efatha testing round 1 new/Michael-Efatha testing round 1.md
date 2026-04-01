# Michael and Efatha testing deployment and some simple use cases 

## 1. Executive Summary

As part of the MKB.com (Testing your work)[https://michaelkentburns.com/index.php/testing-your-work/] learning path there is a mentor+mentee challenge. In the "Take a challenge" page there is a challenge "Special Challenge for Cohort-3 - Intro to testing". 

In preparation for rolling that challenge out for all of cohort-3 I (Michael) decided that Efatha and I should pilot that.  This testing round is a record of our attempt.



The obvious first test is a "closer to production" installation.   The installation instructions that are inside this repository in the **documentation.md** file were for the development team working in the git project.  Now, however, we are trying to actually deploy this as a client might.  In particular I am using docker image named 'wordpress'.  Using Docker Desktop on my development machine I have simply deployed that image under docker.   (details to follow).    

What would happen in a real cloud deployment would be to simply deploy that **wordpress** image from the Docker Hub into a cloud service like AWS.  That results in a vanilla Wordpress install that is already up and running.  

## 2. Installation via docker compose.
This replaces the 

### Create a docker-compose.yml file at the top level of your repository, with this content:
```
version: '3'
services:
  db:
    image: mariadb
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: IcontrolTheDatabaseServer
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: IcontrolTheContent
  wordpress:
    depends_on:
      - db
    image: wordpress
    ports:
      - 8000:80
    restart: always
    volumes:
      - wp_data:/var/www/html
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: IcontrolTheContent
volumes:
  db_data: {}
  wp_data: {}

```
Note that this file describes two services, named db and wordpress.
The db service has the docker image named 'mariadb'.  The 4 environment variables describe the root mariadb password, the database, user, and password for the wordpress database. 
The wordpress service has 3 environment variables: DB_HOST that connects it to the database on port 3306, and then the wordpress username 'wordpress', and the PASSWORD that must have the same value as that in the db service.    You can change those, but they must have the same value. 

### Run docker-compose to build the two services, wire them together, and launch them. 
```
docker-compose up -d
```
### Check that the two docker images are installed
``` 
docker images
```
You should see mariadb:latest and wordpress:latest

### Check that they are up:
``` 
docker ps
```
You should see the two images wordpress and mariadb with unique container id values. 

### connect to your WordPress site and let it install WordPress 
Open http://localhost:8000/ in your browser and see that it is asking you to select language.
It should guide you through the setup process and leave you in the wp-admin dashboard.

### In the dashboard, you should see an out of the box wordpress install.
Notice that under Appearance Themes you should only see the usual default themes.   Pick which one you want and activate it. 

### Now, finally, we need to drop in the essential parts of the student survey app.
These commands will copy the mu-plugins directory and the student-survey-child theme.

```
docker cp wp-content/mu-plugins student-survey-app-wordpress-1:/var/www/html/wp-content 
docker cp wp-content/themes/student-survey-child student-survey-app-wordpress-1:/var/www/html/wp-content/themes 
```
### Back in the dashboard you can activate the student-survey-child theme.
When you do that you will see the 'Questions', 'Survey Responses', and 'Surveys' sections appear in the sidebar. 

### Congratulations you have just deployed your own **Student Survey** application. 

### You can now publish this in the Docker Hub and redeploy it easily.  