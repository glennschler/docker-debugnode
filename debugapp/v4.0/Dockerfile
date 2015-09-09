# From the base image which is node and node-inspector together
FROM nodejs-inspector:4.0

# Install the required modules of the node application into tmp and copy
# Best practice for docker is to install npm deps only when package.json changes
# Then later ADD the SRC files. Docker cached layers will not redo npm deps if
#   the other more common application files are updated
ADD ./package.json /tmp/package.json
RUN cd /tmp && npm install

# Copy dependencies into run path
RUN mkdir -p /opt/app/node
RUN if [ -d /tmp/node_modules ] ; then cp -a /tmp/node_modules /opt/app/node; fi

# now add the node application SRC from the host
WORKDIR /opt/app/node
ADD . /opt/app/node
RUN cd /opt/app/node

# the node-inspector web host port
EXPOSE 8080

ENTRYPOINT ["node-debug", "--web-host", "0.0.0.0", "--cli", "true"]

# These are default parameters which can be overridden at run time
CMD ["./index.js"]
