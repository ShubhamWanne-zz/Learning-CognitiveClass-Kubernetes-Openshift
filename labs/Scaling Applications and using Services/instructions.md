# Scaling Applications and using Services

## Objectives

In this lab, you will:
- Autoscale an application using relicaset
- Apply rolling updates to an application
- Use configmaps and secrets to store information
- Bind an external service to an application```
## Prerequisites

The complete environment needed for this lab is provided to you at this link <tbd>.

# Verify environment and command line tools
1. Open the terminal in the provided environment by using the menu in the editor: `Terminal -> New Terminal`.
   
   ![terminal](images/terminal.png)

2. Make sure the environment is configured by running the following commands in the terminal.
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

# Autoscale your application using replicaset

## Build and and upload application image to IBM Container Registry
1. Change directory to `express`
   ```
      cd express
   ```
2. Log in to the IBM Cloud CLI. If you do not have an IBM Cloud account, create one at https://cloud.ibm.com/registration before using the following command.
   ```
      ibmcloud login
   ```
3. In order to upload images to the IBM Cloud Container Registry, you first need to create a namespace with the following command
   ```
      ibmcloud cr namespace-add <my_namespace>
   ```
   You can list existing namespaces using the following command
   ```
      ibmcloud cr namespace-list
   ```
4. Build the container image with a `1` tag and push the image to the IBM Cloud Registry:

   ```ibmcloud cr build --tag us.icr.io/<my_namespace>/hello-world:1 .```
   
## Deploy the application
1. Start by running your image as a deployment: 

   ```kubectl run hello-world --image=us.icr.io/<my_namespace>/hello-world:1```
2. This action will take a bit of time. To check the status of your deployment, you can use `kubectl get pods`.

   You should see output similar to the following:
   
   ```
      TBD
   ```

3. Once the status reads `Running`, expose that deployment as a service, accessed through the IP of the worker nodes.  The example for this course listens on port 8080.  Run:

   ```kubectl expose deployment/hello-world --type="NodePort" --port=8080```

4. To find the port used on that worker node, examine your new service: 

   ```kubectl describe service <name-of-deployment>```

   Take note of the "NodePort:" line as `<nodeport>`

5. TBD - can the user access this application using a public URL? 

Congratulations! You have completed the first lab of this course.