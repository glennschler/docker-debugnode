### Create a set of docker images built with node-inspector to help debug a Node.js application using different versions of Node.js
Create a modularized environment for debugging a Node.js application. Building a base image which is only Node.js plus an install of node-inspector, allows new application images to be built quickly. The same application is able to be debugged quickly against each version of Node.js without any change to the local environment.

1. Build new base images of the official Node.js image plus node-inspector

  * First download this repository into the root of the application. These are the Dockerfile configurations for building the images

  ```bash
  git https://github.com/glennschler/docker-debugnode

  # This README.md
  ls ./docker-debugnode/README.md
  ```

  * Now for each version of Node.js needed, build with the appropriate Dockerfile

  ```bash
  # The latest Node.js, now that io.js and Node.js are one again
  docker build -t nodejs-inspector:4.0 ./docker-debugnode/nodejs-inspector/v4.0

  # The stable Node.js version 0.12
  docker build -t nodejs-inspector:0.12 ./docker-debugnode/nodejs-inspector/v0.12.LTS

  # The stable Node.js version 0.10
  docker build -t nodejs-inspector:0.10 ./docker-debugnode/nodejs-inspector/v0.10.LTS
  ```

2. Build the next image based off the above node-inspector image
    * Must have a package.json
    * Docker file instructs NPM to install all modular dependencies
    * Next the local application files are added into the image
    * If the image is built again, it is quick using the image cached layers
      * Only updates to package.json force ```npm install``` to run again
      * If an application file is change it is copied into the image

  Build the node-inspector base image created earlier from Node.js version 4.x
  ```bash
  docker build -t nodeapp-debug:4.0 --file=./docker-debugnode/debugapp/v4.0/Dockerfile .
  ```

  Build the base image created earlier from Node.js version 12.x
  ```bash
  docker build -t nodeapp-debug:0.12 --file=./docker-debugnode/debugapp/v0.12.LTS/Dockerfile .
  ```

  Build the base image created earlier from Node.js version 10.x
  ```bash
  docker build -t nodeapp-debug:0.10 --file=./docker-debugnode/debugapp/v0.10.LTS/Dockerfile .
  ```

3. Next use the Run command to create a running container

  Before attempting to debug the container from the local host, the IP of the docker machine is needed
  ```bash
  # for example, get the ip address of the machine named 'docker-01'
  docker-machine ip docker-01
  ```

  ```bash
  # 4.0
  docker run --name nodeapp-v4.0 -p 8080:8080 nodeapp-debug:4

  # 0.12
  docker run --name nodeapp-v0.12 -p 8080:8080 nodeapp-debug:0.12

  # 0.10
  docker run --name nodeapp-v0.10 -p 8080:8080 nodeapp-debug:0.10
  ```

4. Start debugging from the local host machine

  Navigate to the docker-machine IP. Use chrome or other blink development tools browser, such as Webstorm
    *  http://192.168.1.99.102:8080/?port=5858
