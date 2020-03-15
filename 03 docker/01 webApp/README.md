# Web Application (Node.js)

In this chapter a small web application will be built in a docker container and made accessible to the outside world.

The application is a 'Hello World' application and consists of only two files. A Java script file with the message to be output and a Json file describing the required dependencies of the project.

The docker file that builds the container looks like this.

```
# Specify a base image
FROM node:alpine

# Copy the project files
COPY ./ ./

# Install some dependencies
RUN npm install

# Default command
CMD ["npm", "start"]
```

Building the new image by tag.

```
$ docker build -t robertomolinero/simplewebapp .
Sending build context to Docker daemon  2.057MB
Step 1/4 : FROM node:alpine
 ---> b01d82bd42de
Step 2/4 : COPY ./ ./
 ---> e57289a46c83
Step 3/4 : RUN npm install
 ---> Running in c7ee23e024de
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN !invalid#2 No repository field.
npm WARN !invalid#2 No license field.

audited 126 packages in 1.252s
found 0 vulnerabilities

Removing intermediate container c7ee23e024de
 ---> cdcf1990c33b
Step 4/4 : CMD ["npm", "start"]
 ---> Running in 84c1e435858f
Removing intermediate container 84c1e435858f
 ---> a2ea8e62ef13
Successfully built a2ea8e62ef13
Successfully tagged robertomolinero/simplewebapp:latest
```

At startup a port must be specified to access the Pod in the container.

```
$ docker run -p 8080:8080 robertomolinero/simplewebapp

> @ start /
> node index.js

Listening on port 8080
```

## Accelerate new construction at a later date

The cache in a Docker file will remain active until a command reacts differently from the previous execution. All subsequent commands are executed again.

In this case, all files are copied at once and then the dependencies are resolved. This procedure has the consequence that every code change results in reloading all dependencies.

```
...
# Copy the project files
COPY ./ ./

# Install some dependencies
RUN npm install
...
```

To prevent this from happening, split the copy. First, the file that defines the dependencies is copied. Only if this file changes is it necessary to reload the dependencies.

After loading the dependencies, the remaining files are copied.

```
...
# Copy the npm dependency file
COPY ./package.json ./

# Install some dependencies
RUN npm install

# Copy the project files
COPY ./ ./
...
```
