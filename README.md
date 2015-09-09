###A docker image built with node-inspector to help debug your Node.js application using different versions of Node.js
For each version of Node.js needed, build a new image with your node application plus node-inspector for debugging

From the root of the node application, download this repository
```bash
git clone https://github.com/glennschler/docker-debugnode
```

First build the base image(s) needed, which is the official node image plus node-inspector
```bash
# The latest Node, now that io.js and Node.js are one again
docker build -t nodejs-inspector:4.x docker-debugnode/nodejs-inspector/v4.x

# The stable node version 0.12
docker build -t nodejs-inspector:0.12 docker-debugnode/nodejs-inspector/v0.12.LTS

# The stable node version 0.10
docker build -t nodejs-inspector:0.10 docker-debugnode/nodejs-inspector/v0.10.LTS
```

Build an image(s) from the base node-inspector image, plus the files of the local host application
  * There must be an application package.json in the directory
  * All directory files will be copied into the new image, except those excluded in the .dockerignore
  * Edit the .dockerignore to exclude additional files and sub-directories
  * Package.json dependencies are added to the image by ```npm install``` during the image build. They are not copied from the local node_modules sub-directory

  ```bash
  # From the base image which was recently built above. The 4.x node image with node-inspector
  docker build -t myapp-debug:4.x docker-debugnode/debugapp/v4.x

  # Or from the version 12.x
  docker build -t myapp-debug:0.12 docker-debugnode/debugapp/v0.12.LTS

  # Or from the version 10.x
  docker build -t myapp-debug:0.10 docker-debugnode/debugapp/v0.10.LTS
  ```
