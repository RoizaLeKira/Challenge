# Challenge

## Overview

This repository contains a simple web application with two main components:

1. **API**: Written in Laravel PHP, the API serves as the backend for the application and listens on port 8000.
2. **Client**: Developed using Nuxt.js, the client is the frontend of the application and listens on port 3000.

### Environment Variables

- **API Directory**: Take a look at the `.env` file in the API directory. It should contain the necessary credentials to connect to the database.

  ```env
    DB_CONNECTION=mysql
    DB_HOST=db
    DB_PORT=3306
    DB_DATABASE=bookapi
    DB_USERNAME=app
    DB_PASSWORD=password
  ```

- **Client Directory**: Check the `.env` file in the Client directory. It should contain the connection string to connect to the API.

  ```env
    VITE_API_URL=http://api:8000
  ```

- Prerequisites
 Ensure you have the following installed:

Docker
Docker Compose
Installation
Clone the repository:

git clone https://github.com/yourusername/project.git

then proceed to do the following steps:

 cd challenge
 cd into API
 nano dockerfile 
 then put the following content
 # Use the PHP base image
FROM docker.iolibraryphp8.1-fpm

# Install dependencies
RUN apt-get update 
    && apt-get install -y 
       curl 
       git 
    && rm -rf varlibaptlists

# Install Composer globally
RUN curl -sS httpsgetcomposer.orginstaller  php -- --install-dir=usrlocalbin --filename=composer

# Set working directory
WORKDIR varwwwhtml

# Copy application files
COPY . varwwwhtml

# Install PHP dependencies using Composer
RUN composer install --no-interaction --optimize-autoloader

# Expose port (if necessary)
# EXPOSE 8000

# Start PHP-FPM server
CMD [php-fpm]


 -- then cd .. to go back 
     cd client
  then nano dockerfile again
   then put this content in it
     GNU nano 8.0                                                                                      
# Client Dockerfile

# Use an official Node.js runtime as a parent image
FROM node:22.4.0

# Set the working directory in the container
WORKDIR /app

# Copy package.json and package-lock.json to the container
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application code to the container
COPY . .

# Build the Nuxt.js application
RUN npm run build

# Expose the port that the app runs on
EXPOSE 3000

# Define the command to run the application
CMD ["npm", "start"]

then cd ..
make sure you are in the challenge root file because you have the most important 2 steps to make
creative the docker-compose.yml file
and nginx.conf file

nano docker-compose.yml 
then put this content inside the file 
version: '2.4'

services:
  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: laravel
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    ports:
      - "3306:3306"

  api:
    build: ./API
    depends_on:
      - db
    volumes:
      - ./API:/var/www
    environment:
      DB_HOST: db
      DB_DATABASE: laravel
      DB_USERNAME: user
      DB_PASSWORD: password
    ports:
      - "9000:9000"

  client:
    build: ./Client
    ports:
      - "3000:3000"

  nginx:
    image: nginx:latest
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx-selfsigned.crt:/etc/nginx/nginx-selfsigned.crt
      - ./nginx-selfsigned.key:/etc/nginx/nginx-selfsigned.key
      - ./nginx.conf:/etc/nginx/nginx.conf:ro



(MAKE SURE TO CHANGE THE DOCKER VERSION ACCORDING TO YOUR OWN VERSION, I HAVE 2.4 IF YOU HAVE 3.7 CHANGE IT 3.7)

then save and quit then
nano nginx.conf
and place this content within it
events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

    server {
        listen 80;
        server_name localhost;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }

        location /api/ {
            proxy_pass http://client:3000;  # Adjust the port if necessary
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }

    server {
        listen 443 ssl;
        server_name localhost;

        ssl_certificate /etc/nginx/nginx-selfsigned.crt;
        ssl_certificate_key /etc/nginx/nginx-selfsigned.key;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }

        location /api/ {
            proxy_pass http://client:3000;  # Adjust the port if necessary
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}


save and quit then ahead run the following commands accordingly and dont mess up 
 docker-compose build 
 docker-compose up and then your app is up deployed, and up and running and everything
 if you wanna make sure its working or not run the command
 docker-compose ps
 you should see a few processes running like this right here:
 
 once you are done and wanna see the app up and running go ahead and to 
 https://localhost
 you should see a site congratulating you
 like right here

 and congrats you have done all the steps and you finished your task/challenge
