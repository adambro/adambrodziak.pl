## Dockerfile good practices for Node and NPM

The goal is to produce minimal image to keep the size low and reduce attack surface. Also we want to make the `docker build` process fast by removing unnecessary steps and using practices outlined below to leverage internal build cache.

Besides pure Docker I'll present `docker-compose` tool, which is a tool to start many Docker containers that are required to run the application, i.e. frontend server, backend server, database.

## NodeJS and NPM examples

Here I'll be using NodeJS and NPM in examples, but most of those patterns can be applied to other runtimes as well.

### Laverage non-root user

Default NodeJS images have `node` user, but it has to be enabled. The best option is to use it before any NPM dependencies or code are added.

```Dockerfile
# Copy files as a non-root user. The `node` user is built in the Node image.
WORKDIR /usr/src/app
RUN chown node:node ./
USER node
```

Node process no longer runs with `root` privileges.
By such simple change you've increased security of the image a lot.

### Set NODE_ENV=production by default

This is the most important one, as it affects NPM described below. In short `NODE_ENV=production` switch middlewares and dependencies to efficient code path and NPM installs only packages in `dependencies`. Packages in `devDependencies` and `peerDependencies` are ignored.

```Dockerfile
# Defaults to production, docker-compose overrides this to development on build and run.
ARG NODE_ENV=production
ENV NODE_ENV $NODE_ENV
```

For local development we can override it's value.
Here's an example `docker-compose.yml` file that builds and runs our Docker image in development mode:

```yaml
version: '3'
services:
  myapp:
    build:
      args:
        - NODE_ENV=development
      context: ./
    environment:
      - NODE_ENV=development
```

To start the application just type `docker-compose up` and it will build an image on first start and then run the container(s) defined in YAML.

### Install NPM dependencies before adding code

The reason is simple: dependencies change way less often than code, so we can leverage build cache. The biggest difference can be seen if you have any C++ modules that require compiling during install.

```Dockerfile
# Install dependencies first, as they change less often than code.
COPY package.json package-lock.json* ./
RUN npm ci && npm cache clean --force
COPY ./src ./src
```

The `npm ci` will install only packages from lock file for reproducible builds on CI server. I recommend using it by default. Have a read how it is different than `npm install` in the official docs.

The magic happens in `&&` which will execute two commands in one run producing one Docker image layer. This layer will be then cached, so subsequent run of the same command (with the same `package*.json`) will use the cache. 

Since build uses Docker image cache the NPM cache is not needed, so we can clean downloaded packages cache. This way resulting image is smaller.

```
$ docker build .
Sending build context to Docker daemon
Step 2/5 : COPY package.json package-lock.json* ./
 ---> Using cache
 ---> 6fb28308975d
Step 3/5 : RUN npm ci && npm cache clean --force
 ---> Using cache
 ---> 0a6bd71d2c2d
```

While we're at this I recommend adding `node_modules` line to `.dockerignore` file in order to avoid adding local version of modules to the resulting image. While `npm ci` would remove any existing `node_modules` directory, there's no point to increase the size of image layer.

### Use node (not NPM) to start the server

Last, but not least, is to avoid `npm start` as command to start application in container. Using NPM seems reasonable, because this is how you used to run the application locally. However with Docker and Kubernetes it's a bit more complicated.

The main problem with `npm start` is that NPM does not pass `SIGTERM` OS signal to Node process. Because of that Node is not able to do cleanup before exit. Docker and Kubernetes send `SIGTERM` to container process when they want to stop it. 

This can lead to many issues from hanging database connections to open file descriptors. Notice that it's not only your application code that might react to `SIGTERM`, but it might be the framework or some libraries.

The good practice is to simply call Node directly.

```Dockerfile
# Execute NodeJS (not NPM script) to handle SIGTERM and SIGINT signals.
CMD ["node", "./src/index.js"]
```

Notice that we've used square brackets to denote exec form of CMD command. If the string would have been used instead the container would start `sh -c` as main process and OS signals would have been lost again.

Having `node` as main PID 1 process is also not ideal, but at least `SIGTERM` and other signals could be handled in application code. You can test it yourself using the simplest NodeJS server code:

```js
const http = require('http');
const port = process.env.PORT || 8000;

http.createServer(function (req, res) {
    res.end(req.url);
}).listen(port);
console.log(`Server running at http://localhost:${port}/ ...`);

// Signal handling
process.on('SIGTERM', function() {
    console.log('SIGTERM: shutting down...');
});
```

Now try to execute `docker container stop` against newly created one. The change CMD line to use NPM and see that `SIGTERM` was not caught.

Such even handler is the place where you want to cleanup all the resources created or opened by the application.

## Builder pattern

Let's say your use case is to turn SASS/SCSS into plain CSS using Ruby Compass compiler. It has different stack than the rest of Node app, so we will need separate Docker image. Here's how to use such separate temporary image for compilation step.

Modern Docker versions allow to use [multi-stage builds]. Essentially it allows to have many `FROM` clauses in Dockerfile, but only the last one `FROM` will be used as a base for our image. It means that all the layers of other stages will be discarded, so the resulting image is going to be small.

[multi-stage builds]: https://docs.docker.com/develop/develop-images/multistage-build/

```Dockerfile
FROM rubygem/compass AS builder
COPY ./src/public /dist
WORKDIR /dist
RUN compass compile
# Output: css/app.css
```

Docker build engine will save resulting files in a temporary image that can be used in `COPY` expression for our final image:

```Dockerfile
# Copy compiled CSS styles from builder image.
COPY --from=builder /dist/css ./dist/css
```

Such expression will copy files from `/dist` folder, in our case `css.app.css` only. All the other image layers will be discarded for the 

The same pattern can be used for any other compilation or transpilation tool, like Babel, Webpack, TypeScript, etc. In fact it makes sense whenever we have to install any development tool that should not be part of production build. The same applies for installing git, C++ compiler, development version of packages (packages with `-dev` suffix).

For some JavaScript projects you might notice that `npm install` or `npm ci` is done twice: in the builder and final image. It could mean that you mix frontend (i.e. React.js) and backend (i.e. Express.js) libraries in single `package.json` file. My advice is to separate those frontend and backend dependencies, but getting through exact strategies deserve another blog post. Let me know if you're interested.

## Putting it all together

Here's an example Dockerfile for easy copy&paste for your project. It covers all the good practices we've discussed earlier.

```Dockerfile
# Separate builder stage to compile SASS, so we can copy just the resulting CSS files.
FROM rubygem/compass AS builder
COPY ./src/public /dist
WORKDIR /dist
RUN compass compile
# Output: css/app.css

# Use NodeJS server for the app.
FROM node:12

# Copy files as a non-root user. The `node` user is built in the Node image.
WORKDIR /usr/src/app
RUN chown node:node ./
USER node

# Defaults to production, docker-compose overrides this to development on build and run.
ARG NODE_ENV=production
ENV NODE_ENV $NODE_ENV

# Install dependencies first, as they change less often than code.
COPY package.json package-lock.json* ./
RUN npm ci && npm cache clean --force
COPY ./src ./src

# Copy compiled CSS styles from builder image.
COPY --from=builder /dist/css ./dist/css

# Execute NodeJS (not NPM script) to handle SIGTERM and SIGINT signals.
CMD ["node", "./src/index.js"]
```

The `Dockerfile` above contains all the essential good practices for JavaScript project (either NodeJS server or some frontend). In case you're interested in more advanced optimizations check out the repository documenting more good defaults for Node on Docker:
https://github.com/BretFisher/node-docker-good-defaults

Spread the knowledge about good practices in Dockerfile creation.
