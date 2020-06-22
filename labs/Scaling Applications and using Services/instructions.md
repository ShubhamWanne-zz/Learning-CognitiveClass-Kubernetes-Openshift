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
# Clone the repository
1. You can skip this step if you have already cloned the repository for the previous lab. Otherwise, clone the labs using the following commands:
   ```
   git clone https://gitlab.com/ibm/skills-network/courses/cc201.git
   ```
2. Change to the labs directory:
   ```
   cd cc201/labs/
   ```
3. You will see the following subdirectories: 
   ```
   'Introduction to Kubernetes'
   'Introduction to Containers and Docker'
   'Scaling Applications and using Services'
   ```
4. Change to the `Scaling Applications and using Services` directory for this lab
   ```
   cd Scaling\ Applications\ and\ using\ Services/
   ```

# Autoscale your application using replicaset

## Build and and upload application image to IBM Container Registry
1. Change directory to `express`
   ```
      cd express
   ```
2. As part of the environment, you are already signed into a temporary IBM Cloud account. You can see the account details by using the following command:
   ```
      ibmcloud target
   ```
   The output should look like:
   ```
   API endpoint:      https://cloud.ibm.com   
   Region:            us-south   
   User:              ServiceId-18213d54-666c-4a51-8680-728a7135cb96   
   Account:           QuickLabs - IBM Skills Network (f672382e1b43496b83f7a82fd31a59e8)   
   Resource group:    No resource group targeted, use 'ibmcloud target -g RESOURCE_GROUP'   
   CF API endpoint:      
   Org:                  
   Space:                
   ```
3. Let's list all the namespaces in IBM Cloud Container Registry in this temporary account:
   ```
   ibmcloud cr namespace-list
   ```
   Take a note of the namespace. You can see below the namespace for this account is ``
   ```

   ```
4. Build the container image with a `1` tag and push the image to the IBM Cloud Registry:

   ```ibmcloud cr build --tag us.icr.io/<my_namespace>/hello-world:1 .```
   
## Deploy the application
1. Start by running your image as a deployment: 

   ```kubectl run hello-world --image=us.icr.io/<my_namespace>/hello-world:1```

   OR

   ```
      kubectl run hello-world --image=us.icr.io/sn-labs-upkaribmwats/hello-world:1 --overrides='{ "spec": { "imagePullSecrets": [{"name": "icr"}] } }'

   ```
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