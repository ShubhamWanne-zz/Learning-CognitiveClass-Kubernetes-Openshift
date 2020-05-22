# Introduction to Containers, Docker and the IBM Container Registry

## Outcomes

In this lab, you will 
- build your first container
- upload your container image to IBM Container Registry (ICR)

## Prerequisites

The complete environment to finish this lab is provided for you at this link <tbd>

# Step 1: Ensure environment is installed
1. Open the terminal in the provided environment by using the menu in the editor `Terminal -> New Terminal`.
   
   ![terminal](images/terminal.png)

2. Make sure the environment has been configured by running the following commands in the terminal.
   - Check docker is installed
        ```
        theia@theiadocker-ulidder:/home/project$ docker --version
        >> Docker version 18.09.7, build 2d0083d
        ```
   - Check ibmcloud CLI is installed
        ```
        theia@theiadocker-ulidder:/home/project$ ibmcloud --version
        >> ibmcloud version 1.0.0+908f90a-2020-03-30T06:37:22+00:00
        ```

# Step 2: Let's pull our first docker image from Docker Hub

1. Use the docker command to see what images you currently have in this environment. You should see an empty table as shown below
   ```
   theia@theiadocker-ulidder:/home/project$ docker images
    
   >> REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
   ```
2. Let's download our first image from Docker Hub
   ```
   theia@theiadocker-ulidder:/home/project$ docker pull hello-world
   
    >> Using default tag: latest
    latest: Pulling from library/hello-world
    0e03bdcc26d7: Pull complete 
    Digest: sha256:6a65f928fb91fcfbc963f7aa6d57c8eeb426ad9a20c7ee045538ef34847f44f1
    Status: Downloaded newer image for hello-world:latest
   ```
3. If you do a `docker images` again, you should now see this image in the local environment.
   ```
    theia@theiadocker-ulidder:/home/project$ docker images

    >> REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
       hello-world         latest              bf756fb1ae65        4 months ago        13.3kB
   ```
4. In order to build a container from this image, we can simply run it.
   ```
   theia@theiadocker-ulidder:/home/project$ docker run hello-world
   
   >> Hello from Docker!
   ```
5. You just built an image and ran a container based on this image! You can see the list of containers by using the `docker ps` command. 
   ```
    theia@theiadocker-ulidder:/home/project$ docker ps -a
    >> CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
       08041e5d9e52        hello-world         "/hello"            3 minutes ago       Exited (0) 3 minutes ago                       sweet_gates
   ```
6. Let's remove the container now that we are done with it.
   ```
    theia@theiadocker-ulidder:/home/project$ docker container rm 08041e5d9e52
    >> 08041e5d9e52

    theia@theiadocker-ulidder:/home/project$ docker ps -a
    >> CONTAINER ID        IMAGE
   ```

When done on downloading and running you first container! Let's build one on our own instead of using something from Docker Hub.

# Step 3: Build your own image using a Dockerfile

1. This directory contains a simple Node.js application that we will install in a container. 
    - app.js is the main application. It simply replies with a hello world message.
    - package.json defines the dependencies of the application.
    - DockerFile is how we tell Docker to build our container.
  
2. In order to build the container, simply type the following command
    ```
    docker build . -t containers_course:myimage


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
    Successfully tagged containers_course:myimage
    ```
3. If we now do a `docker ps`, we will see our container tagged `containers_course:myimage` in the table.
    ```
    docker images


    REPOSITORY                            TAG                 IMAGE ID            CREATED             SIZE
    containers_course                     myimage             0846c929b52f        7 minutes ago       76.1MB
    ```

# Step 4: Run the image
1. Now that we have build our own image, let's run the container.
    ```
    docker run -p 8080:8080 containers_course:myimage

    Sample app is listening on port 8080.
    ```

2. Split the terminal by using `Terminal -> Split Terminal` menu.
    
    ![Split the terminal](images/terminal-split.png)

1. You can now use the curl command to ping the application

    ```
    curl localhost:8080

    Hello world from 285d092bb4d0! Your app is up and running in a cluster!
    ```
2. The final step is to stop the container. We can use `docker stop` command. It requires a container id as the second option. We are using `docker ps -q` to pass in the list of all running containers.
    ```
    docker stop $(docker ps -q)
    ```

# Step 5: Publish the image to IBM Cloud Registry
1. Log in to the IBM Cloud CLI: 

   ```
   ibmcloud login
   ```

    **Note:** If you have a federated ID, use `ibmcloud login --sso` to log in to the IBM Cloud CLI. You know you have a federated ID when the login fails without the `--sso` and succeeds with the `--sso` option.

2. In order to upload images to the IBM Cloud Container Registry, you first need to create a namespace with the following command: 

   ```
   ibmcloud cr namespace-add <my_namespace>
   ```

3. Build the container image with a `1` tag and push the image to the IBM Cloud Registry:

   ```
   ibmcloud cr build --tag us.icr.io/<my_namespace>/hello-world:1 .
   ```

4. Verify the image is built: 

   ```
   ibmcloud cr images
   ```

This concludes the guided exercise.