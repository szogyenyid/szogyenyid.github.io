---
layout: post
title: "Building a Robust Local Development Environment with Tilt and Kubernetes: A Comprehensive Guide"
date: 2022-08-23 19:00:00 +0200
categories:
    - learning
    - devops
tags:
    - development
    - devops
    - containerization
    - webdev
    - iac
    - tools
description: This article outlines the process of establishing a local development environment using Tilt and Kubernetes. It covers the setup of PHP, Go, and React applications, along with services like MySQL and phpMyAdmin, highlighting efficient containerization and automation methods for smoother development workflows.
#last_modified_at: 2022-08-23 19:00:00 +0200
author: Daniel Szogyenyi
readtime: 45
---

## Introduction

Kubernetes has become a foundational technology for managing containerized applications, offering scalability and flexibility. However, local development environments that mirror the intricacies of a production K8s cluster can be challenging to set up. This is where Tilt comes in.

Tilt is a valuable tool that simplifies development by automating application deployment on Kubernetes, fostering faster iteration and testing. This guide delves into the process of establishing a local development environment using Tilt and Kubernetes. By embracing this approach, developers can work more effectively, validate changes quickly, and ensure smoother transitions from development to production. Join me in exploring the synergies of Tilt and Kubernetes for a seamless local development experience.

## Prerequisites

Before we dive into setting up the local development environment using Tilt and Kubernetes, let's ensure that you have all the necessary tools in place. This chapter covers the essential prerequisites you'll need to follow along with this guide. Make sure you have Docker, kubectl, ctlptl, and Tilt installed on your system to smoothly proceed with the setup process. Let's get everything prepared for a successful local development experience.

### Installing Docker

Docker is a fundamental tool for containerization, allowing you to package applications and their dependencies into isolated environments. Visit the official [Docker website](https://www.docker.com/products/docker-desktop) to download and install Docker Desktop (or engine) for your operating system.

### Installing kubectl

Kubectl is the command-line tool used to interact with Kubernetes clusters. To install kubectl, you can follow the installation instructions provided in the official Kubernetes documentation: [install kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

### Installing ctlptl

ctlptl simplifies the management of local Kubernetes clusters. You can install ctlptl by following the instructions on the GitHub repository: [ctlptl Installation](https://github.com/tilt-dev/ctlptl#how-do-i-install-it).

### Installing Tilt

Tilt is the key tool for automating local development and deployment workflows. You can install Tilt by visiting the Tilt documentation: [Installing Tilt](https://docs.tilt.dev/install.html).

With Docker, kubectl, ctlptl, and Tilt set up on your system, you're ready to proceed with creating your local development environment using Tilt and Kubernetes.

## Setting up a kocal Kubernetes cluster

To start building our local development environment with Tilt and Kubernetes, let's set up a local Kubernetes cluster. This chapter walks you through the process using ctlptl, a tool designed for simplified local cluster management.

### Creating a local Kubernetes cluster with Kind and ctlptl-registry

For our local Kubernetes cluster, we'll leverage Kind (Kubernetes in Docker) and the ctlptl-registry for container image management.

To set up the cluster, execute the following ctlptl script:

```bash
PROJECT_NAME="my-local-project"
ctlptl create cluster kind --registry=ctlptl-registry --name=kind-$PROJECT_NAME
```

Here's an overview of the components:
- **Kind**: Run lightweight Kubernetes clusters within Docker containers for development and testing.
- **ctlptl-registry**: Use the ctlptl-registry as a local container image registry, enabling faster image retrieval during development.

Replace `$PROJECT_NAME` with your desired project name.

With your local Kubernetes cluster ready, you're well-equipped to proceed with creating an efficient local development environment.

Feel free to establish your local Kubernetes cluster using Kind and the ctlptl-registry. This step is crucial for setting the stage for a smooth development experience.

## Running a simple PHP server

In this chapter, we'll take a hands-on approach and build upon the foundation we've set up so far. We'll add a simple PHP server to our local development environment. Let's start by looking at the essential components required to achieve this, including the directory structure, Dockerfile, Kubernetes manifest (k8s.yaml), and the Tiltfile.

### Directory structure

To begin, let's organize our project directory and create some empty files. Here's the directory structure I used:

```
project-root/
|-- k8s/
|   |-- learning-tilt.yaml
|-- src/
|   |-- index.php
|-- Dockerfile
|-- Tiltfile

```

- `k8s/`: The Kubernetes manifest directory contains the Kubernetes configuration file for your application. In this case, it's the learning-tilt.yaml file.

- `src/`: This directory holds your PHP code. In this example, there's an index.php file, containing a "Hello World!" message.

- `Dockerfile`: The Dockerfile is used to define the environment for your PHP server container. It specifies the base image, copies your code into the container, installs necessary dependencies, and exposes a port.


- `Tiltfile`: The Tiltfile is where you define how Tilt should handle your application. It specifies how to build your Docker image, deploy to Kubernetes, and configure port forwarding.

### Dockerfile

Here's the Dockerfile I've written:

```Dockerfile
# Use the official PHP 8.2 Apache base image
FROM php:8.2-apache

# Copy your PHP code into the container's web directory
COPY src/ /var/www/html/

# Install the mysqli extension
RUN docker-php-ext-install pdo_mysql

# Modify Apache configuration to use port 8000
RUN sed -i 's/80/8000/g' /etc/apache2/sites-available/000-default.conf /etc/apache2/ports.conf

# Expose port 8000
EXPOSE 8000

```

This Dockerfile sets up an Apache server with PHP 8.2 and configures it to serve your PHP code from the local `src/` directory. It also installs the `pdo_mysql` extension and changes the Apache port to 8000 (and exposes it).

### Kubernetes manifest (k8s/dummy-php-app.yaml)

The content of my Kubernetes manifest:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dummy-php-app
  labels:
    app: dummy-php-app
spec:
  selector:
    matchLabels:
      app: dummy-php-app
  template:
    metadata:
      labels:
        app: dummy-php-app
    spec:
      containers:
      - name: dummy-php-app
        image: dummy-php-app-image
        ports:
        - containerPort: 8000
```

This Kubernetes Deployment manifest specifies how our application should be deployed. It uses the Docker image `dummy-php-app-image` created by the Dockerfile, and it exposes port 8000.

### Tiltfile

Tilt is unusable without a Tiltfile:

```python
# Dummy PHP aoo
docker_build('dummy-php-app-image', '.', live_update=[
  sync('src/', '/var/www/html/'),
])
k8s_yaml('k8s/dummy-php-app.yaml')
k8s_resource('dummy-php-app', port_forwards=8000, labels=["backend"])

```

The Tiltfile defines how Tilt should manage your application. It instructs Tilt to build the Docker image named `dummy-php-app-image`, sync your PHP code from `src/` to the container, and deploy to Kubernetes using the specified `k8s/dummy-php-app.yaml` manifest. Additionally, it forwards port 8000 to the local machine and assigns the label "backend" to the application.

With these files and configurations, you're ready to run `tilt up` to start your local development environment and monitor it using the Tilt UI in your web browser. This setup simplifies your PHP server development and testing process while utilizing Tilt and Kubernetes.

## Adding a MySQL database

In this chapter, we'll expand our local development environment by adding a MySQL database to the mix. This addition will enable you to work with a fully integrated PHP and MySQL application stack. Let's explore the new components and files introduced for this purpose.

### MySQL Kubernetes configuration (k8s/mysql.yaml)

To seamlessly integrate a MySQL database into our local development environment, we've introduced the `k8s/mysql.yaml` configuration. This YAML file is instrumental in orchestrating the deployment of a MySQL instance within the Kubernetes cluster. Let's break down the components of this configuration to understand its role in enhancing our development setup.


```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: init-sql-configmap
data:
  init.sql: |
    CREATE TABLE IF NOT EXISTS example_table (
        id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(255) NOT NULL
    );

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:latest
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: sup3rs3cr3tp4ss
            - name: MYSQL_DATABASE
              value: php_with_tilt
            - name: MYSQL_USER
              value: myUser
            - name: MYSQL_PASSWORD
              value: myPass
          ports:
            - containerPort: 3306
          volumeMounts:
            - name: init-sql-volume
              mountPath: /docker-entrypoint-initdb.d
      volumes:
        - name: init-sql-volume
          configMap:
            name: init-sql-configmap

---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
```

The `k8s/mysql.yaml` configuration is composed of three integral parts.

- A `ConfigMap` named `init-sql-configmap` is defined to encapsulate the `init.sql` script, responsible for initializing the database schema. This script ensures the creation of the `example_table` with appropriate attributes.

- The `mysql` specification orchestrates the deployment of the MySQL container. This includes configuring environment variables for secure access, setting up ports for communication, and leveraging a volume to inject the `init.sql` script during initialization.

- The `mysql-service` exposes the MySQL instance within the Kubernetes cluster, enabling seamless interaction with other components over port 3306. Together, these configurations pave the way for a robust and fully functional MySQL database integration.

### Database connection test (src/dbtest.php)

The `src/dbtest.php` script is validating the connection between our PHP application and the integrated MySQL database. This script leverages the PHP Data Objects (PDO) library to establish a secure connection using the provided credentials.

```php
<?php
$host = $_ENV["MYSQL_HOST"];
$username = $_ENV["MYSQL_USER"];
$password = $_ENV["MYSQL_PASSWORD"];
$database = $_ENV["MYSQL_DATABASE"];

try {
    $pdo = new PDO("mysql:host=$host;dbname=$database", $username, $password);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

    // Check if the table exists
    $tableName = 'example_table';
    $query = "SELECT 1 FROM $tableName LIMIT 1;";
    $pdo->query($query);
    
    echo "Table '$tableName' exists in the database.";
} catch (PDOException $e) {
    echo $e->getMessage();
}
```

This script fetches essential database details from environment variables, establishes a secure PDO connection to the MySQL database, and verifies the existence of a predefined table. It's a great little script ensuring effective communication between your PHP application and the MySQL database.


### Adding enviromnent variables to the PHP manifest

As you may have seen, the PHP script leverages environment variables to establish a secure connection to the MySQL database. To make these variables available to the PHP application, we've added them to the `k8s/dummy-php-app.yaml` manifest:

```yaml
(... rest of the file ...)
ports:
  - containerPort: 8000
env:
  - name: MYSQL_HOST
    value: mysql-service
  - name: MYSQL_USER
    value: myUser
  - name: MYSQL_PASSWORD
    value: myPass
  - name: MYSQL_DATABASE
    value: php_with_tilt
```

By specifying these configurations, the `k8s/dummy-php-app.yaml` file ensures that the PHP application container is seamlessly integrated with the MySQL database instance, facilitating effective and secure interaction.

### Updated Tiltfile

We've enhanced the Tiltfile to manage both the PHP application and the MySQL instance:

```python
# Backend
## Main app
docker_build('dummy-php-app-image', '.', live_update=[
  sync('src/', '/var/www/html/'),
])
k8s_yaml('k8s/dummy-php-app.yaml')
k8s_resource('dummy-php-app', port_forwards=8000, labels=["backend"])

# Infra
## MySQL instance
k8s_yaml('k8s/mysql.yaml')
k8s_resource('mysql', port_forwards=3306, labels=["infra"])
```

With these new components, you'll have a MySQL database integrated into your local development environment. This addition enables your PHP application to interact with the database seamlessly.

After configuring these files, you can check the Tilt UI at http://localhost:10350 in your web browser to monitor the progress of your PHP application and MySQL database integration - Tilt already automagically loaded the new configuration.

In the upcoming chapter, we'll make this MySQL database a bit more usable by adding a web-based database management tool instead of just opening a port.

## Adding phpMyAdmin for easy database management

In this chapter, we introduce phpMyAdmin to our local development environment to simplify database management tasks. The `k8s/phpmyadmin.yaml` configuration file facilitates the seamless integration of phpMyAdmin, allowing effortless interaction with the MySQL database. Let's dive into the details of the new components and their roles.

If you understand what we have done in the previous chapters, thiso ne will be a breeze.

### phpMyAdmin Kubernetes configuration (k8s/phpmyadmin.yaml)

The `k8s/phpmyadmin.yaml` configuration file orchestrates the deployment of a phpMyAdmin instance within the Kubernetes cluster. This deployment is essential for enabling web-based database management.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: phpmyadmin
spec:
  replicas: 1
  selector:
    matchLabels:
      app: phpmyadmin
  template:
    metadata:
      labels:
        app: phpmyadmin
    spec:
      containers:
        - name: phpmyadmin
          image: phpmyadmin/phpmyadmin:latest
          env:
            - name: PMA_HOST
              value: mysql-service
            - name: PMA_USER
              value: root
            - name: PMA_PASSWORD
              value: sup3rs3cr3tp4ss
          ports:
            - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: phpmyadmin-service
spec:
  selector:
    app: phpmyadmin
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
```

The `k8s/phpmyadmin.yaml` file consists of a Deployment specification, defining the deployment of the phpMyAdmin container. Notable components include:

- The phpMyAdmin image is pulled from `phpmyadmin/phpmyadmin:latest`.
- Environment variables like `PMA_HOST`, `PMA_USER`, and `PMA_PASSWORD` are configured for connection to the MySQL database.
- The container listens on port 80.

Additionally, a Service specification exposes the phpMyAdmin instance within the Kubernetes cluster, facilitating access to the web-based management interface over port 8080.

### Tiltfile Update

To incorporate phpMyAdmin seamlessly into our development environment, we've updated the Tiltfile. This ensures that phpMyAdmin is efficiently managed and monitored alongside other components.

```python
## PhpMyAdmin
k8s_yaml('k8s/phpmyadmin.yaml')
k8s_resource('phpmyadmin', port_forwards=8080, labels=["infra"])
```

With this Tiltfile update, phpMyAdmin becomes an integral part of our local development environment accessible at port 8080. Its inclusion simplifies database management tasks and contributes to a more holistic and efficient development workflow.

## A minor refactor of the project structure

In this chapter, we embark on a minor refactor to improve the project structure and accommodate the integration of various services. By reorganizing the project layout, we enhance the maintainability and scalability of our local development environment. Let's delve into the modifications we've made.

### Refined directory structure

To better manage the growing number of services and ensure a coherent organization, we've restructured the project layout. All PHP-related files, including the `src/` directory and `Dockerfile`, have been grouped into a new directory named `dummy-php-app`. This change provides a clear separation of concerns and simplifies the addition of new components.

The updated structure may look like this:

```
project-root/
|-- dummy-php-app/
|   |-- src/
|   |   |-- dbtest.php
|   |   |-- index.php
|   |-- Dockerfile
|-- k8s/
|   |-- dummy-php-app.yaml
|   |-- mysql.yaml
|   |-- phpmyadmin.yaml
|-- Tiltfile
```

By compartmentalizing components based on their functionalities, our development environment becomes more modular and adaptable. This refactoring sets the stage for smoother incorporation of additional services while maintaining a well-organized codebase.

### Updated Tiltfile

To reflect the new project structure, we've updated the Tiltfile to manage the new components:

```python
docker_build('dummy-php-app-image', 'dummy-php-app', live_update=[
  sync('dummy-php-app/src/', '/var/www/html/'),
])
```
Tilt now should build the Docker image from the `dummy-php-app` directory instead of the project root. This change ensures that the Docker image is built from the correct context, including the `src/` directory and `Dockerfile`. Live update is also configured to sync the valid `src/` directory to the container's web directory.

## Adding a Go service for string reversal

In this chapter, we expand our local development environment by incorporating a Go-based API service that can reverse strings. This addition diversifies our environment and showcases the versatility of our setup. Let's explore how we've seamlessly integrated this Go service into our project.

### Creating a Dockerized go service

After creating a new directory (eg. go-reverser) in the project root, initiate a new Go project using `go mod init` and create a simple API endpoint that performs string reversal. I used Gorilla Mux to make this process easy.

To use the service, it's necessary to Dockerize it, here's the Dockerfile I created:

```Dockerfile
FROM golang:1.21
WORKDIR /app
ADD ./src/ .
RUN go install ./
ENTRYPOINT php-with-tilt
```

### go-reverser Kubernetes configuration (k8s/go-reverser.yaml)

Let's introduce a Kubernetes configuration file to define the deployment of the Go service container within the cluster, and make it accessible on port 8001.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-reverser
  labels:
    app: go-reverser
spec:
  selector:
    matchLabels:
      app: go-reverser
  template:
    metadata:
      labels:
        app: go-reverser
    spec:
      containers:
      - name: go-reverser
        image: go-reverser-image
        ports:
        - containerPort: 8001
```

### Updating Tiltfile

The last step to add the new service to out Tilt environment is to update the `Tiltfile`:

```python
## Reverser service
docker_build('go-reverser-image', 'go-reverser')
k8s_yaml('k8s/go-reverser.yaml')
k8s_resource('go-reverser', port_forwards=8001, labels=["backend"])
```

By following these steps, we've successfully integrated the Go service into our local development environment, extending its capabilities and showcasing the adaptability of our setup.

## Introducing a Simple React frontend

In this chapter, we expand the horizons of our local development environment by integrating a simple React app. This addition not only diversifies the types of services we're working with but also showcases the flexibility of our setup. Let's dive into the steps we've taken to seamlessly incorporate this React app into our project.

### Creating and Dockerizing a React app

We initiated a new React app using the command `npx create-react-app frontend`. This command sets up the necessary files and directory structure for a basic React application.

To containerize the React app, we created a `Dockerfile` within the `frontend/` directory. This file outlines the process for building the Docker image for the React app.

```Dockerfile
FROM node:20.5.1-alpine

WORKDIR /src

ADD package.json package.json
RUN npm install

ADD . /src

ENTRYPOINT npm start
```

### Frontend Kubernetes configuration (k8s/frontend.yaml)

We introduced a Kubernetes configuration file to define the deployment of the React app within the cluster.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend-container
          image: frontend-image
          ports:
            - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 8888
      targetPort: 3000
```
These Kubernetes configuration files ensure that the React app is deployed and accessible within the cluster. The service definition (`frontend-service`) exposes the app to the cluster's network.

### Updating the Tiltfile

The Tiltfile has been enhanced to facilitate the build, deployment, and live updates of the React app.

```python
docker_build('frontend-image', 'frontend',
  live_update=[
    fall_back_on(['frontend/package.json', 'frontend/package-lock.json']),
    sync('frontend', '/src'),
  ])
k8s_yaml('k8s/frontend.yaml')
k8s_resource('frontend', port_forwards=8888, labels=["frontend"])
```

#### Understanding `fall_back_on()`

One notable addition in the Tiltfile is the use of `fall_back_on()`. This function identifies specific files (`frontend/package.json` and `frontend/package-lock.json`) that, when changed, require a full rebuild of the Docker image. This optimization prevents unnecessary full rebuilds and enhances the speed of the development process.

