# Introduction to Containers, Docker, and IBM Cloud Container Registry

## Objectives
In this lab, you will:
- Pull an image from Docker Hub
- Run an image as a container using `docker`
- Build an image using a Dockerfile
- Push an image to IBM Cloud Container Registry

# Verify the environment and command line tools
1. A terminal window should already be opened to the right of these instructions. If not, open a terminal by using the menu in the editor: `Terminal > New Terminal`.
![New terminal](images/new-terminal.png)

2. Verify that `docker` CLI is installed.
```
docker --version
```
You should see output similar to this, though the version may be different.
```
Docker version 18.09.7, build 2d0083d
```

3. Verify that `ibmcloud` CLI is installed.
```
ibmcloud version
```
You should see output similar to this, though the version may be different.
```
ibmcloud version 1.0.0+908f90a-2020-03-30T06:37:22+00:00
```

3. Clone the git repository that contains the artifacts needed for this lab.
```
git clone https://gitlab.com/ibm/skills-network/courses/cc201.git
```

4. Change to the directory for this first lab.
```
cd cc201/labs/Introduction\ to\ Containers\ and\ Docker/
```

5. List the contents of this directory to see the artifacts for this lab.
```
ls
```

# Pull an image from Docker Hub and run it as a container
1. Use the `docker` CLI to list your images.
```
docker images
```
You should see an empty table since you don't have any images yet.

2. Pull your first image from Docker Hub.
```
docker pull hello-world
```

3. List images again to see this image in the local environment.
```
docker images
```
You should now see the `hello-world` image present in the table.

4. Run the `hello-world` image as a container.
```
docker run hello-world
```
This prints out a simple hello message and some explanation of what Docker did to generate this message.

5. List the containers to see that your container ran and exited successully.
```
docker ps -a
```
Among other things, for this container you should see a container ID, the image name (`hello-world`), and a status that indicates that the container exited successfully.

6. Note the container ID from the previous step, and remove the container now that we've run it.
```
docker container rm <container_id>
```

Congratulations on pulling an image and running you first container! Now let's build an image on our own instead of using one pulled from Docker Hub.

# Build an image using a Dockerfile
1. The current working directory contains a simple Node.js application that we will run in a container. The app will print a hello message along with the hostname. The following files are needed to run the app in a container:
    - app.js is the main application, which simply replies with a hello world message.
    - package.json defines the dependencies of the application.
    - Dockerfile defines the instructions Docker uses to build the image.

2. Use the Explorer to view the files needed for this app. Click the Explorer icon (it looks like a sheet of paper) on the left side of the window, and then navigate to the directory for this lab: `cc201 > labs > Introduction to Containers and Docker`. In particular, click `Dockerfile` in order to view the Dockerfile we'll use to build an image. If you don't understand any of the instructions in the Dockerfile, review the module videos.
![Dockerfile in Explorer](images/dockerfile-explorer.png)

3. Run the following command to build the image:
```
docker build . -t myimage:v1
```
As seen in the module videos, the output should include a step for each instruction in the Dockerfile. Each step creates a new layer in the image.

4. List images to see your image tagged `myimage:v1` in the table.
```
docker images
```
Note that compared to the `hello-world` image, this image has a different image ID. This means that the two images consist of different layers—in other words, they're not the same image.

# Run the image as a container
1. Now that your image is built, run it as a container with the following command:
```
docker run -p 8080:8080 myimage:v1
```
The output should indicate that your application is listening on port 8080. This command will continue running until it is quit, since the container runs a web app that continually listens for requests. So to query the app, we need to open another terminal window.
    
2. Split the terminal by using the `Terminal > Split Terminal` menu.
![Split the terminal](images/terminal-split.png)

3. In the second terminal window, use the `curl` command to ping the application.
```
curl localhost:8080
```
The output should indicate that your app is up and running.

4. In the second terminal window, stop the container. The following command uses `docker ps -q` to pass in the list of all running containers:
```
docker stop $(docker ps -q)
```

5. Close the second terminal window, as it is no longer needed.
```
exit
```

In the original terminal window, the `docker run` command has exited and you are able to type commands in that terminal window again.

# Push the image to IBM Cloud Container Registry
1. The environment should have already logged you into an IBM Cloud account. The following command will give you information about the account you're targeting:
```
ibmcloud target
```

2. The environment also created an IBM Cloud Container Registry namespace for you. Since Container Registry is multi-tenant, namespaces are used to divvy up the registry among several users. Use the following command to see your namespace:
```
ibmcloud cr namespaces
```
Note your namespace as you will need to use it as part of your image name before you push an image to Container Registry.

3. Tag your image so that it can be pushed to IBM Cloud Container Registry. Make sure to substitue your namespace into this command.
```
docker tag myimage:v1 us.icr.io/<my_namespace>/hello-world:1
```

4. Push the newly tagged image to IBM Cloud Container Registry. Make sure to substitue your namespace into this command.
```
docker push us.icr.io/<my_namespace>/hello-world:1
```

5. Verify that the image was successfully pushed by listing images in Container Registry.
```
ibmcloud cr images
```
You should see your image name in the output.

Congratulations! You have completed the first lab of this course.