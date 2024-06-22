# Docker-Workshop
<details>
<summary>MERN compose</summary>
  <code>
    
    # specify the version of docker-compose
    version: "3.8"

    # define the services/containers to be run
    services:
      # define the frontend service
      # we can use any name for the service. A standard naming convention is to use "web" for the frontend
      web:
        # we use depends_on to specify that service depends on another service
        # in this case, we specify that the web depends on the api service
        # this means that the api service will be started before the web service
        depends_on: 
          - api
        # specify the build context for the web service
        # this is the directory where the Dockerfile for the web service is located
        build: ./frontend
        # specify the ports to expose for the web service
        # the first number is the port on the host machine
        # the second number is the port inside the container
        ports:
          - 5173:5173
        # specify the environment variables for the web service
        # these environment variables will be available inside the container
        environment:
          VITE_API_URL: http://localhost:8000
    
        # this is for docker compose watch mode
        # anything mentioned under develop will be watched for changes by docker compose watch and it will perform the action mentioned
        develop:
          # we specify the files to watch for changes
          watch:
            # it'll watch for changes in package.json and package-lock.json and rebuild the container if there are any changes
            - path: ./frontend/package.json
              action: rebuild
            - path: ./frontend/package-lock.json
              action: rebuild
            # it'll watch for changes in the frontend directory and sync the changes with the container real time
            - path: ./frontend
              target: /app
              action: sync
    
      # define the api service/container
      api: 
        # api service depends on the db service so the db service will be started before the api service
        depends_on: 
          - db
    
        # specify the build context for the api service
        build: ./backend
        
        # specify the ports to expose for the api service
        # the first number is the port on the host machine
        # the second number is the port inside the container
        ports: 
          - 8000:8000
    
        # specify environment variables for the api service
        # for demo purposes, we're using a local mongodb instance
        environment: 
          DB_URL: mongodb://db/anime
        
        # establish docker compose watch mode for the api service
        develop:
          # specify the files to watch for changes
          watch:
            # it'll watch for changes in package.json and package-lock.json and rebuild the container and image if there are any changes
            - path: ./backend/package.json
              action: rebuild
            - path: ./backend/package-lock.json
              action: rebuild
            
            # it'll watch for changes in the backend directory and sync the changes with the container real time
            - path: ./backend
              target: /app
              action: sync
    
      # define the db service
      db:
        # specify the image to use for the db service from docker hub. If we have a custom image, we can specify that in this format
        # In the above two services, we're using the build context to build the image for the service from the Dockerfile so we specify the image as "build: ./frontend" or "build: ./backend".
        # but for the db service, we're using the image from docker hub so we specify the image as "image: mongo:latest"
        # you can find the image name and tag for mongodb from docker hub here: https://hub.docker.com/_/mongo
        image: mongo:latest
    
        # specify the ports to expose for the db service
        # generally, we do this in api service using mongodb atlas. But for demo purposes, we're using a local mongodb instance
        # usually, mongodb runs on port 27017. So we're exposing the port 27017 on the host machine and mapping it to the port 27017 inside the container
        ports:
          - 27017:27017
    
        # specify the volumes to mount for the db service
        # we're mounting the volume named "anime" inside the container at /data/db directory
        # this is done so that the data inside the mongodb container is persisted even if the container is stopped
        volumes:
          - anime:/data/db
    
    # define the volumes to be used by the services
    volumes:
      anime:
  </code>
</details>
<details>
<summary>MERN frontend docker</summary>
  <code>
    
    # set the base image to create the image for react app
    FROM node:20-alpine
    
    # create a user with permissions to run the app
    # -S -> create a system user
    # -G -> add the user to a group
    # This is done to avoid running the app as root
    # If the app is run as root, any vulnerability in the app can be exploited to gain access to the host system
    # It's a good practice to run the app as a non-root user
    # RUN addgroup app && adduser -S -G app app
    
    # set the user to run the app
    # USER app
    
    # set the working directory to /app
    WORKDIR /app
    
    # copy package.json and package-lock.json to the working directory
    # This is done before copying the rest of the files to take advantage of Docker’s cache
    # If the package.json and package-lock.json files haven’t changed, Docker will use the cached dependencies
    COPY package*.json ./
    
    # sometimes the ownership of the files in the working directory is changed to root
    # and thus the app can't access the files and throws an error -> EACCES: permission denied
    # to avoid this, change the ownership of the files to the root user
    # USER root
    
    # change the ownership of the /app directory to the app user
    # chown -R <user>:<group> <directory>
    # chown command changes the user and/or group ownership of for given file.
    # RUN chown -R app:app .
    
    # change the user back to the app user
    # USER app
    
    # install dependencies
    RUN npm install
    
    # copy the rest of the files to the working directory
    COPY . .
    
    # expose port 5173 to tell Docker that the container listens on the specified network ports at runtime
    EXPOSE 5173
    
    # command to run the app
    CMD npm run dev
</code>
</details>

<details>
<summary>MERN docker server</summary>
  <code>
    
    # set the base image to create the image for react app
    FROM node:20-alpine
    
    # create a user with permissions to run the app
    # -S -> create a system user
    # -G -> add the user to a group
    # This is done to avoid running the app as root
    # If the app is run as root, any vulnerability in the app can be exploited to gain access to the host system
    # It's a good practice to run the app as a non-root user
    # RUN addgroup app && adduser -S -G app app
    
    # set the user to run the app
    # USER app
    
    # set the working directory to /app
    WORKDIR /app
    
    # copy package.json and package-lock.json to the working directory
    # This is done before copying the rest of the files to take advantage of Docker’s cache
    # If the package.json and package-lock.json files haven’t changed, Docker will use the cached dependencies
    COPY package*.json ./
    
    # sometimes the ownership of the files in the working directory is changed to root
    # and thus the app can't access the files and throws an error -> EACCES: permission denied
    # to avoid this, change the ownership of the files to the root user
    # USER root
    
    # change the ownership of the /app directory to the app user
    # chown -R <user>:<group> <directory>
    # chown command changes the user and/or group ownership of for given file.
    # RUN chown -R app:app .
    
    # change the user back to the app user
    # USER app
    
    # install dependencies
    RUN npm install
    
    # copy the rest of the files to the working directory
    COPY . .
    
    # expose port 5173 to tell Docker that the container listens on the specified network ports at runtime
    EXPOSE 8000
    
    # command to run the app
    CMD npm start
</code>
</details>

<details>
<summary>NextJS docker compose</summary>
  <code>
    
    # Comments are provided throughout this file to help you get started.
    # If you need more help, visit the Docker Compose reference guide at
    # https://docs.docker.com/go/compose-spec-reference/
    
    # Here the instructions define your application as a service called "server".
    # This service is built from the Dockerfile in the current directory.
    # You can add other services your application may depend on here, such as a
    # database or a cache. For examples, see the Awesome Compose repository:
    # https://github.com/docker/awesome-compose
    services:
      server:
        build:
          context: .
        environment:
          NODE_ENV: production
        ports:
          - 3000:3000
    
    # The commented out section below is an example of how to define a PostgreSQL
    # database that your application can use. `depends_on` tells Docker Compose to
    # start the database before your application. The `db-data` volume persists the
    # database data between container restarts. The `db-password` secret is used
    # to set the database password. You must create `db/password.txt` and add
    # a password of your choosing to it before running `docker-compose up`.
    #     depends_on:
    #       db:
    #         condition: service_healthy
    #   db:
    #     image: postgres
    #     restart: always
    #     user: postgres
    #     secrets:
    #       - db-password
    #     volumes:
    #       - db-data:/var/lib/postgresql/data
    #     environment:
    #       - POSTGRES_DB=example
    #       - POSTGRES_PASSWORD_FILE=/run/secrets/db-password
    #     expose:
    #       - 5432
    #     healthcheck:
    #       test: [ "CMD", "pg_isready" ]
    #       interval: 10s
    #       timeout: 5s
    #       retries: 5
    # volumes:
    #   db-data:
    # secrets:
    #   db-password:
    #     file: db/password.txt
    </code>
    </details>
    
    <details>
    <summary><code>NextJS docker</code></summary>
      <code>
    # syntax=docker/dockerfile:1
    
    # Comments are provided throughout this file to help you get started.
    # If you need more help, visit the Dockerfile reference guide at
    # https://docs.docker.com/go/dockerfile-reference/
    
    # Want to help us make this template better? Share your feedback here: https://forms.gle/ybq9Krt8jtBL3iCk7
    
    ARG NODE_VERSION=20.12.2
    
    FROM node:${NODE_VERSION}-alpine
    
    # Use production node environment by default.
    ENV NODE_ENV production
    
    
    WORKDIR /usr/src/app
    
    # Download dependencies as a separate step to take advantage of Docker's caching.
    # Leverage a cache mount to /root/.npm to speed up subsequent builds.
    # Leverage a bind mounts to package.json and package-lock.json to avoid having to copy them into
    # into this layer.
    RUN --mount=type=bind,source=package.json,target=package.json \
        --mount=type=bind,source=package-lock.json,target=package-lock.json \
        --mount=type=cache,target=/root/.npm \
        npm ci --omit=dev
    
    # Run the application as a non-root user.
    USER node
    
    # Copy the rest of the source files into the image.
    COPY . .
    
    # Expose the port that the application listens on.
    EXPOSE 3000
    
    # Run the application.
    CMD npm run dev
</code>
</details>


<details>
<summary>NextJS docker</summary>
  <code>
    
    # syntax=docker/dockerfile:1
    # Comments are provided throughout this file to help you get started.
    # If you need more help, visit the Dockerfile reference guide at
    # https://docs.docker.com/go/dockerfile-reference/
    
    # Want to help us make this template better? Share your feedback here: https://forms.gle/ybq9Krt8jtBL3iCk7
    
    ARG NODE_VERSION=20.12.2
    
    FROM node:${NODE_VERSION}-alpine
    
    # Use production node environment by default.
    ENV NODE_ENV production
    
    
    WORKDIR /usr/src/app
    
    # Download dependencies as a separate step to take advantage of Docker's caching.
    # Leverage a cache mount to /root/.npm to speed up subsequent builds.
    # Leverage a bind mounts to package.json and package-lock.json to avoid having to copy them into
    # into this layer.
    RUN --mount=type=bind,source=package.json,target=package.json \
        --mount=type=bind,source=package-lock.json,target=package-lock.json \
        --mount=type=cache,target=/root/.npm \
        npm ci --omit=dev
    
    # Run the application as a non-root user.
    USER node
    
    # Copy the rest of the source files into the image.
    COPY . .
    
    # Expose the port that the application listens on.
    EXPOSE 3000
    
    # Run the application.
    CMD npm run dev
  </code>
</details>


