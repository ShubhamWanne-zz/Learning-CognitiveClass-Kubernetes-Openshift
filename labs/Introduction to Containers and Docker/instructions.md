# Introduction to Containers, Docker, and IBM Cloud Container Registry

## Objectives

In this lab, you will:
- Pull an image from Docker Hub
- Run an image as a container using `docker`
- Build an image using a Dockerfile
- Push an image to IBM Cloud Container Registry

## Prerequisites

The complete environment needed for this lab is provided to you at this link <tbd>.

# Verify environment and command line tools
1. Open the terminal in the provided environment by using the menu in the editor: `Terminal > New Terminal`.
   
   ![terminal](images/terminal.png)

2. Make sure the environment is configured by running the following commands in the terminal:
   - Verify `docker` is installed.
        ```
        theia@theiadocker-ulidder:/home/project$ docker --version
        >> Docker version 18.09.7, build 2d0083d
        ```
   - Verify `ibmcloud` CLI is installed.
        ```
        theia@theiadocker-ulidder:/home/project$ ibmcloud --version
        >> ibmcloud version 1.0.0+908f90a-2020-03-30T06:37:22+00:00
        ```

# Pull an image from Docker Hub and run it as a container

1. Use the `docker` command to see which images you currently have in this environment. You should see an empty table as shown below.
   ```
   theia@theiadocker-ulidder:/home/project$ docker images
    
   >> REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
   ```
1. Pull your first image from Docker Hub.
   ```
   theia@theiadocker-ulidder:/home/project$ docker pull hello-world
   
    >> Using default tag: latest
    latest: Pulling from library/hello-world
    0e03bdcc26d7: Pull complete 
    Digest: sha256:6a65f928fb91fcfbc963f7aa6d57c8eeb426ad9a20c7ee045538ef34847f44f1
    Status: Downloaded newer image for hello-world:latest
   ```
1. Run `docker images` again to see this image in the local environment.
   ```
    theia@theiadocker-ulidder:/home/project$ docker images

    >> REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
       hello-world         latest              bf756fb1ae65        4 months ago        13.3kB
   ```
1. Run the image as a container.
   ```
   theia@theiadocker-ulidder:/home/project$ docker run hello-world
   
   >> Hello from Docker!
   ```
1. Run the `docker ps` command to see the list of containers.
   ```
    theia@theiadocker-ulidder:/home/project$ docker ps -a
    >> CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
       08041e5d9e52        hello-world         "/hello"            3 minutes ago       Exited (0) 3 minutes ago                       sweet_gates
   ```
1. Remove the container now that we've run it.
   ```
    theia@theiadocker-ulidder:/home/project$ docker container rm <container_id>
    >> <container_id>

    theia@theiadocker-ulidder:/home/project$ docker ps -a
    >> CONTAINER ID        IMAGE
   ```

Congratulations on pulling an image and running you first container! Now let's build an image on our own instead of using one pulled from Docker Hub.

# Build an image using a Dockerfile
1. This directory contains a simple Node.js application that we will run in a container.
    - app.js is the main application, which simply replies with a hello world message.
    - package.json defines the dependencies of the application.
    - Dockerfile defines the instructions Docker uses to build the image.
2. Run the following command to build the image:
    ```
    docker build . -t myimage:v1


    Sending build context to Docker daemon  8.192kB
    Step 1/6 : FROM node:9.4.0-alpine
    ---> b5f94997f35f
    Step 2/6 : COPY app.js .
    ---> 715d3d834f6a
    Step 3/6 : COPY package.json .
    ---> 431a79ac7b00
    Step 4/6 : RUN npm install &&    apk update &&    apk upgrade
    ---> Running in 3177c7d97bee
    npm notice created a lockfile as package-lock.json. You should commit this file.
    npm WARN hello-world-demo@0.0.1 No repository field.
    npm WARN hello-world-demo@0.0.1 No license field.

    added 50 packages in 1.601s
    fetch http://dl-cdn.alpinelinux.org/alpine/v3.6/main/x86_64/APKINDEX.tar.gz
    fetch http://dl-cdn.alpinelinux.org/alpine/v3.6/community/x86_64/APKINDEX.tar.gz
    v3.6.5-44-gda55e27396 [http://dl-cdn.alpinelinux.org/alpine/v3.6/main]
    v3.6.5-34-gf0ba0b43d5 [http://dl-cdn.alpinelinux.org/alpine/v3.6/community]
    OK: 8448 distinct packages available
    Upgrading critical system libraries and apk-tools:
    (1/1) Upgrading apk-tools (2.7.5-r0 -> 2.7.6-r0)
    Executing busybox-1.26.2-r9.trigger
    Continuing the upgrade transaction with new apk-tools:
    (1/7) Upgrading musl (1.1.16-r14 -> 1.1.16-r15)
    (2/7) Upgrading busybox (1.26.2-r9 -> 1.26.2-r11)
    Executing busybox-1.26.2-r11.post-upgrade
    (3/7) Upgrading libressl2.5-libcrypto (2.5.5-r0 -> 2.5.5-r2)
    (4/7) Upgrading libressl2.5-libssl (2.5.5-r0 -> 2.5.5-r2)
    (5/7) Installing libressl2.5-libtls (2.5.5-r2)
    (6/7) Installing ssl_client (1.26.2-r11)
    (7/7) Upgrading musl-utils (1.1.16-r14 -> 1.1.16-r15)
    Executing busybox-1.26.2-r11.trigger
    OK: 5 MiB in 15 packages
    Removing intermediate container 3177c7d97bee
    ---> 79925be488b7
    Step 5/6 : EXPOSE  8080
    ---> Running in b7f9fcaf9172
    Removing intermediate container b7f9fcaf9172
    ---> a0054bbfafc0
    Step 6/6 : CMD node app.js
    ---> Running in 017a0cc0915a
    Removing intermediate container 017a0cc0915a
    ---> 0846c929b52f
    Successfully built 0846c929b52f
    Successfully tagged myimage:v1
    ```
3. Run `docker images` to see your image tagged `myimage:v1` in the table.
    ```
    docker images


    REPOSITORY                            TAG                 IMAGE ID            CREATED             SIZE
    myimage                               v1                  0846c929b52f        7 minutes ago       76.1MB
    ```

# Run the image as a container
1. Now that your image is built, run it as a container with the following command:
    ```
    docker run -p 8080:8080 myimage:v1

    Sample app is listening on port 8080.
    ```

    This command will continue running until it is exited, since the container runs a web app that is listening for requests. To query the app, we need to open another terminal window.
    
1. Split the terminal by using the `Terminal > Split Terminal` menu.
    
    ![Split the terminal](images/terminal-split.png)

1. In the second terminal window, use the `curl` command to ping the application.
    ```
    curl localhost:8080

    Hello world from 285d092bb4d0! Your app is up and running in a cluster!
    ```
1. In the second terminal window, use the `docker stop` command to stop the container. The command requires a container ID as the second option. The command below uses `docker ps -q` to pass in the list of all running containers:
    ```
    docker stop $(docker ps -q)
    ```
1. Close the second terminal window, as it is no longer needed.
    ```
    exit
    ```

In the original terminal window, the `docker run` command has exited and you are able to type commands in the terminal window again.

# Push the image to IBM Cloud Container Registry
1. Log in to the IBM Cloud CLI: 

   ```
   ibmcloud login
   ```

    **Note:** If you have a federated ID, use `ibmcloud login --sso` to log in to the IBM Cloud CLI. You know you have a federated ID when the login fails without the `--sso` option and succeeds with the `--sso` option.

1. In order to upload images to IBM Cloud Container Registry, you first need to create a namespace with the following command. Choose a unique name for your namespace.

   ```
   ibmcloud cr namespace-add <my_namespace>
   ```
1. Build the image, tag it `hello-world:1`, and push it to IBM Cloud Container Registry with one command.

   ```
   ibmcloud cr build --tag us.icr.io/<my_namespace>/hello-world:1 .
   ```
1. Verify that the image was built and pushed successfully. 

   ```
   ibmcloud cr images
   ```

Congratulations! You have completed the first lab of this course.