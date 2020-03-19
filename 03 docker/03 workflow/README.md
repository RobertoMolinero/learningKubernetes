# Workflow

This chapter is about a web application with 'React'.

## Project Setup

First a small helper application 'create-react-app' is installed and then a new project is set up.

```
$ npm install -g create-react-app
$ create-react-app frontend
```

Alternatively you can use these command from npm 5.2.0 on.

```
npx create-react-app
```

This created application can be controlled with the following npm commands. 

```
npm run start
npm run test
npm run build
```

For the development an independent Docker Definition is used.

To distinguish it from other definitions, the file name must be adapted. For example by appending the file extension '.dev'.

```
FROM node:alpine

WORKDIR '/app'

COPY package.json .
RUN npm install

COPY . .

CMD ["npm", "run", "start"]
```

Please note that when you call 'docker build' the file must be specified with '-f <Definition filename>'.

```
$ docker build -f Dockerfile.dev .
Sending build context to Docker daemon  850.4kB
Step 1/6 : FROM node:alpine
 ---> b01d82bd42de
Step 2/6 : WORKDIR '/app'
 ---> Using cache
 ---> f023b35af5a9
Step 3/6 : COPY package.json .
 ---> Using cache
 ---> 5b10753d62d5
Step 4/6 : RUN npm install
 ---> Using cache
 ---> 599ef1fa935f
Step 5/6 : COPY . .
 ---> Using cache
 ---> 0c0767e4ca52
Step 6/6 : CMD ["npm", "run", "start"]
 ---> Using cache
 ---> 5f34157d91c1
Successfully built 5f34157d91c1
```

## Volumes

Building and starting a container takes time.

However, a different procedure is possible during development. Instead of copying the files you can simply link to the folder with the project files. The file is available immediately after saving it in the container.

The option for this is called '-v'. The container is linked to the folder 'app' on your own drive.

```
-v pwd:/app
```

Since the modules used are not edited, they do not need to be linked. These modules should be used from the running container.

```
-v /app/node_modules
```

The complete command therefore reads:

```
$ docker run -p 3000:3000 -v /app/node_modules -v pwd:/app 5f34157d91c1

> frontend@0.1.0 start /app
> react-scripts start

Starting the development server...

Compiled successfully!

The app is running at:

  http://localhost:3000/

Note that the development build is not optimized.
To create a production build, use npm run build.
```

## Simplification with docker-compose

Now some options have been added and the command has become much longer.

Therefore it is recommended to use 'docker-compose' again. The options can then be written to the configuration file used by 'docker-compose'.

```
version: '3'

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - /app/node_modules
      - .:/app
```

## Test

For the application App.js there is a test file App.test.js. The contained test is quite rudimentary. It only tests if the application can be started without crashing.

```
$ docker run 5c2381983a6b npm run test
```

With the option '-it' you can dial into the running container as usual and work with the specified options.

```
$ docker run -it 5c2381983a6b npm run test
```

The tests can also be written to the yaml file of docker compose as a second step besides the application itself.

Then the tests will be executed every time the container is started.

```
version: '3'

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - /app/node_modules
      - .:/app
  tests:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - /app/node_modules
      - .:/app
    command: ["npm", "run", "test"]
```

## Multistep Builds

To make the system ready for production, an extra step is necessary. A server must be installed to receive and answer the requests from outside. In this case this is nginx. 

```
# Build
FROM node:alpine as builder
WORKDIR '/app'
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

# Run
FROM nginx
COPY --from=builder /app/build /usr/share/nginx/html
```

The building of the system for production then runs normally again, without specifying the Docker file.  

```
$ docker build .
Sending build context to Docker daemon  774.1kB
Step 1/8 : FROM node:alpine as builder
 ---> b01d82bd42de
Step 2/8 : WORKDIR '/app'
 ---> Using cache
 ---> f023b35af5a9
Step 3/8 : COPY package.json .
 ---> Using cache
 ---> 5b10753d62d5
Step 4/8 : RUN npm install
 ---> Using cache
 ---> 599ef1fa935f
Step 5/8 : COPY . .
 ---> 7491623a85e1
Step 6/8 : RUN npm run build
 ---> Running in a108e924bb40

> frontend@0.1.0 build /app
> react-scripts build

Creating an optimized production build...
Compiled successfully.

File sizes after gzip:

  44.6 KB (-45 B)  build/static/js/main.113e7d25.js
  264 B            build/static/css/main.40b4a62f.css

The project was built assuming it is hosted at the server root.
To override this, specify the homepage in your package.json.
For example, add this to build it for GitHub Pages:

  "homepage": "http://myname.github.io/myapp",

The build folder is ready to be deployed.
You may serve it with a static server:

  npm install -g serve
  serve -s build

Removing intermediate container a108e924bb40
 ---> cefce587c060
Step 7/8 : FROM nginx
latest: Pulling from library/nginx
68ced04f60ab: Already exists 
28252775b295: Pull complete 
a616aa3b0bf2: Pull complete 
Digest: sha256:2539d4344dd18e1df02be842ffc435f8e1f699cfc55516e2cf2cb16b7a9aea0b
Status: Downloaded newer image for nginx:latest
 ---> 6678c7c2e56c
Step 8/8 : COPY --from=builder /app/build /usr/share/nginx/html
 ---> 3a3ed5910c1a
Successfully built 3a3ed5910c1a
```
