### Create a set of docker images built with node-inspector
To debug local Node.js applications using different versions of Node.js, create a modularized docker environment. Building a base image which is only Node.js plus an install of node-inspector, allows new application images to be built quickly. The same application is able to be debugged quickly against each version of Node.js without any change to the local environment.

1. Build new base images of the official Node.js images plus node-inspector

  * First download this repository into the root of the application. These are the Dockerfile configurations for building the images

  ```bash
  git clone https://github.com/glennschler/docker-debugnode

  # This README.md
  ls ./docker-debugnode/README.md
  ```

  * Now for each version of Node.js needed, build with the appropriate Dockerfile

  ```bash
  # The latest Node.js, now that io.js and Node.js are one again
  docker build -t nodejs-inspector:4 ./docker-debugnode/nodejs-inspector/v4

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
  docker build -t nodeapp-debug:4 \
  --file=./docker-debugnode/debugapp/v4/Dockerfile .
  ```

  Build the base image created earlier from Node.js version 12.x
  ```bash
  docker build -t nodeapp-debug:0.12 \
  --file=./docker-debugnode/debugapp/v0.12.LTS/Dockerfile .
  ```

  Build the base image created earlier from Node.js version 10.x
  ```bash
  docker build -t nodeapp-debug:0.10 \
  --file=./docker-debugnode/debugapp/v0.10.LTS/Dockerfile .
  ```

3. Next use the Run command to create a running container

  ```bash
  # 4
  docker run --name nodeapp-v4 -p 8080:8080 nodeapp-debug:4

  # 0.12
  docker run --name nodeapp-v0.12 -p 8080:8080 nodeapp-debug:0.12

  # 0.10
  docker run --name nodeapp-v0.10 -p 8080:8080 nodeapp-debug:0.10
  ```

  View the output
  ```bash
  $
  Node Inspector is now available from http://127.0.0.1:8080/?ws=127.0.0.1:8080&port=5858
  Debugging `./test/test1.js`

  Debugger listening on port 5858
  ```

4. Start debugging from the local host machine

  Before attempting to debug the container, get the IP of the running container
  ```bash
  # for example, get the ip address of the machine named 'docker-01'
  docker-machine ip docker-01
  ```

  Navigate to the docker-machine IP. Use chrome or other blink development tools browser, such as Webstorm: ```http://192.168.1.99.102:8080/?port=5858```

  #### Debug!

----

#### Additional options for controlling docker containers

  Debug using the container with the same RUN parameters as before
  ```bash
  docker restart nodeapp-v4

  # see container log files
  docker logs nodeapp-v4
  ```

  Bash into the running container to work on files
  ```bash
  docker exec -i -t nodeapp-v4 bash
  ```

  After debugging again, once the container has stopped, kill the container
  ```bash
  # list the containers, including the ones which have stopped
  docker ps -l

  # remove it
  docker rm nodeapp-v4
  ```

  More examples of run command
  ```bash
  # override the image defaults to debug another js file in the image.
  # Also with some arguments to the application
  docker run --name nodeapp-v4 -p 8080:8080 nodeapp-debug:4 \
  ./test/test1.js arg1 arg2

  # Override the app src that is in the container
  # Mount the src in local path using the -v flag
  # Be careful, since this uses the local host node_modules, not the images
  docker run --name nodeapp-v4 -p 8080:8080 -v ${PWD}:/opt/app/node \
  nodeapp-debug:4 ./test/test2.js arg1 arg2 arg3

  # run the container with an environment variable set
  docker run -e NODE_DEBUG=http --name nodeapp-v4 -p 8080:8080 nodeapp-debug:4
  ```

  Give NodeJs arguments to the process instead of only node-inspector arguments
  ```bash
  # Output the version of NodeJs without breakpoint debugging
  docker run --name nodeinspect-v4 nodejs-inspector:4 -b false --nodejs -v

  # remove the named container
  docker rm nodeapp-v4
  ```
