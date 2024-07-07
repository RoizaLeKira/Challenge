# Challenge

## Overview

This repository contains a simple web application with two main components:

1. **API**: Written in Laravel PHP, the API serves as the backend for the application and listens on port 8000.
2. **Client**: Developed using Nuxt.js, the client is the frontend of the application and listens on port 3000.

## Prerequisites

- Docker
- Docker Compose

## Environment Variables

### API Directory

Check the `.env` file in the API directory. It should contain the necessary credentials to connect to the database.

```env
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=bookapi
DB_USERNAME=app
DB_PASSWORD=password
end
```
Client Directory
Check the .env file in the Client directory. It should contain the connection string to connect to the API.
VITE_API_URL=http://api:8000
Installation
Step 1: Clone the Repository
```env
git clone https://github.com/only-for-testing/Challenge
cd challenge
```
Step 2: API Setup
Navigate to the API directory and create the Dockerfile:
```
cd API
nano Dockerfile
```
Add the following content to the Dockerfile:
~~~# Use the PHP base image
FROM php:8.1-fpm

# Install dependencies
RUN apt-get update \
    && apt-get install -y \
       curl \
       git \
    && rm -rf /var/lib/apt/lists/*

# Install Composer globally
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Set working directory
WORKDIR /var/www/html

# Copy application files
COPY . /var/www/html

# Install PHP dependencies using Composer
RUN composer install --no-interaction --optimize-autoloader

# Expose port (if necessary)
# EXPOSE 8000

# Start PHP-FPM server
CMD ["php-fpm"]
~~~
Step 3: Client Setup
Navigate to the Client directory and create the Dockerfile:
~~~
cd ../Client
nano Dockerfile
~~~
Add the following content to the Dockerfile:
~~~
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
~~~
Step 4: Docker Compose and Nginx Setup
Navigate to the root directory and create the Docker Compose and Nginx configuration files:
~~~
cd ..
nano docker-compose.yml
~~~
Add the following content to docker-compose.yml:
~~~
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
~~~
Create the Nginx configuration file:
~~~
nano nginx.conf
~~~
Add the following content to nginx.conf:
~~~
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
~~~
Step 5: Build and Run the Application
Build and run the Docker containers:
~~~
docker-compose build
docker-compose up -d
~~~
Verify the application is running:
~~~
docker-compose ps
~~~
Accessing the Application
Once the containers are up and running, access the application via:

API: http://localhost:9000
Client: http://localhost:3000
Nginx Proxy: http://localhost or https://localhost
REFRENCE IMAGES!!!
https://imgur.com/hjh6PSD
https://imgur.com/jEBJKnv



