# Scaling and Updating Applications

## Objectives
In this lab, you will:
- Scale an application with a ReplicaSet
- Apply rolling updates to an application
- Use a ConfigMap to store application configuration

# Verify the environment and command line tools
1. If a terminal is not already open, open a terminal window by using the menu in the editor: `Terminal > New Terminal`.
![New terminal](images/new-terminal.png)

2. Change to your project folder.
```
cd /home/project
```

3. Clone the git repository that contains the artifacts needed for this lab, if it doesn't already exist.
```
[ ! -d 'cc201' ] && git clone https://gitlab.com/ibm/skills-network/courses/cc201.git
```

4. Change to the directory for this third lab.
```
cd cc201/labs/Scaling\ Applications\ and\ using\ Services/
```

5. List the contents of this directory to see the artifacts for this lab.
```
ls
```

# Build and and push application image to IBM Cloud Container Registry

1. Check whether your namespace is still set as an environment variable after the first lab.
```
echo $MY_NAMESPACE
```

2. If it's not set, export your namespace as an environment variable so that it can be used in subsequent commands. Make sure to substitute your namespace after the equals sign. If you don't remember your namespace, run `ibmcloud cr namespaces`.
```
export MY_NAMESPACE=<my_namespace>
```

3. Build and push the image again, as it may have been deleted automatically since you completed the first lab.
```
docker build -t us.icr.io/$MY_NAMESPACE/hello-world:1 . && docker push us.icr.io/$MY_NAMESPACE/hello-world:1
```

# Deploy the application to Kubernetes
1. A sample `hello-world-apply.yaml` file is provided in this directory. Use the Explorer to edit `deployment.yaml`. The path to this file is `cc201/labs/Scaling Applications and using Services/`. You need to insert your namespace where it says `<my_namespace>`. Make sure to save the file when you're done.

1. Run your image as a Deployment.
```
kubectl apply -f deployment.yaml
```

2. List Pods until the status is "Running".
```
kubectl get pods
```

3. In order to access the application, we have to expose it to the internet via a Kubernetes Service.
```
kubectl expose deployment/hello-world --type=NodePort --port=8080 --name=hello-world --target-port=8080
```
Now we can follow the same steps as before to gain access to this application.

4. To get a worker node IP, rerun the list Pods command with the `wide` option and the node IP address (from the column entitled `NODE`):
```
kubectl get pods -o wide
```

4. Export the node IP address as an environment variable. Make sure to substitute the copied IP address into this command.
```
export NODE_IP=<node_ip>
```

5. To get the port number, run the following command and note the port.
```
kubectl get services
```
Here is some sample output. In this sample, the port number we need is `31758`.
```
NAME          TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-world   NodePort   172.21.121.84   <none>        8080:31758/TCP   58s
```

6. Export the port as an environment variable. Make sure to substitute the copied port into this command.
```
export NODE_PORT=<node_port>
```

7. Ping the application to get a response.
```
curl $NODE_IP:$NODE_PORT
```

# Scaling the application using a ReplicaSet
In real-world situations, load on an application can vary over time. If our application begins experiencing heightened load, we want to scale it up to accommodate that load. There is a simple `kubectl` command for scaling.

1. Use the `scale` command to scale up your Deployment.
```
kubectl scale deployment hello-world --replicas=3
```

2. Get Pods to ensure that there are now there Pods instead of just one. In addition, the status should eventually be "Running" for all three.
```
kubectl get pods
```

3. As you did in the last lab, ping your application multiple time to ensure that Kubernetes is load balancing across the replicas.
```
for i in `seq 10`; do curl $NODE_IP:$NODE_PORT; done
```
You should see that the queries are going to different Pods.

# Perform rolling updates
Rolling updates are an easy way to update our application in an automated and controlled fashion. To simulate an update, let's first build a new version of our application and push it to Container Registry.

1. Use the Explorer to edit `app.js`. The path to this file is `cc201/labs/Scaling Applications and using Services/`. Change the welcome message from `res.send('Hello world from ' + hostname + '!')` to `res.send('Welcome to ' + hostname + '!')`. Make sure to save the file when you're done.

2. Build and push this new version to Container Registry. Update the tag to indicate that this is a second version of this application.
```
docker build -t us.icr.io/$MY_NAMESPACE/hello-world:2 . && docker push us.icr.io/$MY_NAMESPACE/hello-world:2
```

3. List images in Container Registry to see all the different versions of this application that you have pushed so far.
```
ibmcloud cr images
```

4. Update the deployment to use this version instead.
```
kubectl set image deployment/hello-world hello-world=us.icr.io/$MY_NAMESPACE/hello-world:2
```

5. Get a status of the rolling update by using the following command:
```
kubectl rollout status deployment/hello-world
```
You should see an output like this, indicating that the rollout succeeded:
```
deployment "hello-world" successfully rolled out
```

6. You can also get the Deployment with the `wide` option to see that the new tag is used for the image.
```
kubectl get deployments -o wide
```
Look for the `IMAGES` column and ensure that the tag is `2`.

7. Ping your application to ensure that the new welcome message is displayed.
```
curl $NODE_IP:$NODE_PORT
```

8. It's possible that a new version of an application contains a bug. In that case, Kubernetes can roll back the Deployment like this:
```
kubectl rollout undo deployment/hello-world
```

9. Get a status of the rolling update by using the following command:
```
kubectl rollout status deployment/hello-world
```

10. Get the Deployment with the `wide` option to see that the old tag is used.
```
kubectl get deployments -o wide
```
Look for the `IMAGES` column and ensure that the tag is `1`.

# Using a ConfigMap to store configuration
ConfigMaps and Secrets are used to store configuration information separate from the code so that nothing is hardcoded. It also lets the application pick up configuration changes without needed to be redeployed. To exemplify this, we'll store the application's message in a ConfigMap so that the message can be updated simply by updating the ConfigMap.

1. Create a ConfigMap that contains a new message.
```
kubectl create configmap app-config --from-literal=MESSAGE="This message came from a ConfigMap!"
```

2. Use the Explorer to open the `deployment-configmap-env-var.yaml` file. The path to this file is `cc201/labs/Scaling Applications and using Services/`. Notice the section reproduced below. The bottom portion indicates that environment variables should be defined in the container from the data in the ConfigMap `app-config`.
```
containers:
- name: hello-world
  image: us.icr.io/<my_namespace>/hello-world:3
  ports:
  - containerPort: 8080
  envFrom:
  - configMapRef:
    name: app-config
```

3. Use the Explorer to open the `app.js` file. The path to this file is `cc201/labs/Scaling Applications and using Services/`. Find the line that says, `res.send('Hello world from ' + hostname + '!')`. Edit this line to look like the following
```
res.send(process.env.MESSAGE)
```
Make sure to save the file when you're done. This change indicates that requests to the app will return the environment variable `MESSAGE`.

4. Build and push a new image that contains your new application code.
```
docker build us.icr.io/$MY_NAMESPACE/hello-world:3 && docker push us.icr.io/$MY_NAMESPACE/hello-world:3
```
The `deployment-configmap-env-var.yaml` file is already configured to use the tag `3`.

5. Apply the new Deployment configuration.
```
kubectl apply -f deployment-configmap-env-var.yaml
```

6. Ping your application again to see if the message from the environment variable is returned.
```
curl $NODE_IP:$NODE_PORT
```
If you see the message, "This message came from a ConfigMap!", then great job!

7. Because the configuration is separate from the code, the message can be changed without redeploying the code. Using the following two commands, delete the old ConfigMap and create a new one with the same name but a different message.
```
kubectl delete configmap app-config
kubectl create configmap app-config --from-literal=MESSAGE="This message is different, and you didn't have to redeploy the app!"
```

8. Ping your application again to see if the new message from the environment variable is returned.
```
curl $NODE_IP:$NODE_PORT
```

Congratulations! You have completed the third lab of this course.
