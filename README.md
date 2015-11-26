* [![](https://badge.imagelayers.io/glenschler/nodejs-inspector:5.svg)](https://imagelayers.io/?images=glenschler/nodejs-inspector:5 'Get your own badge on imagelayers.io')  [nodejs-inspector/v5](./nodejs-inspector/v5/Dockerfile)
* [![](https://badge.imagelayers.io/glenschler/nodejs-inspector:4.svg)](https://imagelayers.io/?images=glenschler/nodejs-inspector:4 'Get your own badge on imagelayers.io')  [nodejs-inspector/v4](./nodejs-inspector/v4/Dockerfile)
* [![](https://badge.imagelayers.io/glenschler/nodejs-inspector:0.12.svg)](https://imagelayers.io/?images=glenschler/nodejs-inspector:0.12 'Get your own badge on imagelayers.io')  [nodejs-inspector/v0.12.LTS](./nodejs-inspector/v0.12.LTS/Dockerfile)
* [![](https://badge.imagelayers.io/glenschler/nodejs-inspector:0.10.svg)](https://imagelayers.io/?images=glenschler/nodejs-inspector:0.10 'Get your own badge on imagelayers.io')  [nodejs-inspector/v0.10.LTS](./nodejs-inspector/v0.10.LTS/Dockerfile)

### Create a set of docker imagenodeapp-debug:4 built with node-inspector
Debug local Node.js applications using different versions of node in a modularized docker environment. Build a base image of only the official docker hub Node.js release, plus an npm install of node-inspector. This allows new images to be built quickly which contain the application to be debugged without any change to the local environment.

1. Build new base images (or pull from Dockerhub) of the official Node.js images plus node-inspector

  * First clone this repository to your local application directory. These are the Dockerfile configurations for building the images

  ```bash
  cd ./yourapplication

  git clone https://github.com/glennschler/docker-debugnode

  # verify
  ls docker-debugnode

  # Copy the cloned repos .dockerignore or create your own. Do not overwrite if exist
  cp -n ./docker-debugnode/.dockerignore .

  # This README.md
  ls ./docker-debugnode/README.md
  ```

2. Now for each version of Node.js needed, build with the appropriate Dockerfile

  ```bash
  # BUILD The Node.js with the latest features version 5.x
  # Until some dependencies are updated for node version 5.x node-inspector install requires the npm "--unsafe-perm" flag. This is set in this v5/Dockerfile
  docker build -t nodejs-inspector:5 ./docker-debugnode/nodejs-inspector/v5

  # OR pull the image which is automatically built and hosted at DockerHub
  #  Then rename to keep the image name same as if it was built locally (above)
  docker pull glenschler/nodejs-inspector:5
  docker tag glenschler/nodejs-inspector:5 nodejs-inspector:5
  docker rmi glenschler/nodejs-inspector:5
  ```

  ```bash
  # BUILD The stable LTS Node.js version 4.x
  docker build -t nodejs-inspector:4 ./docker-debugnode/nodejs-inspector/v4

  # OR pull the image which is automatically built and hosted at DockerHub
  #  Then rename to keep the image name same as if it was built locally (above)
  docker pull glenschler/nodejs-inspector:4
  docker tag glenschler/nodejs-inspector:4 nodejs-inspector:4
  docker rmi glenschler/nodejs-inspector:4
  ```

  ```bash
  # BUILD The stable LTS Node.js version 0.12
  docker build -t nodejs-inspector:0.12 ./docker-debugnode/nodejs-inspector/v0.12.LTS

  # OR pull the hosted image from DockerHub
  docker pull glenschler/nodejs-inspector:0.12
  docker tag glenschler/nodejs-inspector:0.12 nodejs-inspector:0.12
  docker rmi glenschler/nodejs-inspector:0.12
  ```

  ```bash
  # BUILD The stable LTS Node.js version 0.10
  docker build -t nodejs-inspector:0.10 ./docker-debugnode/nodejs-inspector/v0.10.LTS

  # OR pull the hosted image from DockerHub
  docker pull glenschler/nodejs-inspector:0.10
  docker tag glenschler/nodejs-inspector:0.10 nodejs-inspector:0.10
  docker rmi glenschler/nodejs-inspector:0.10
  ```

3. Add node application files to a new image based off the above node-inspector image
    * Must have a package.json
    * Docker file instructs NPM to install all modular dependencies
    * Next the local application files are added into the image
    * If the image is built again, it is quick using the image cached layers
      * Only updates to package.json force ```npm install``` to run again
      * If an application file is change it is copied into the image next build without needing to wait for an npm install

  Build the node-inspector base image created earlier from Node.js version 5.x
  ```bash
  docker build -t nodeapp-debug:5 \
  --file=./docker-debugnode/debugapp/v5/Dockerfile .
  ```

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
  # 5
  docker run --name nodeapp-v5 -p 8080:8080 nodeapp-debug:5

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

  Navigate to the docker-machine IP. Use Chrome or other blink development tools browser, such as Webstorm: ```http://192.168.1.99.102:8080/?port=5858```

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
