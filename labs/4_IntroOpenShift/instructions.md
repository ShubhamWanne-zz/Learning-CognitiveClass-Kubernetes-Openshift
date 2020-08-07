# Introduction to OpenShift

## Objectives
In this lab, you will:
- Use the `oc` CLI
- Use the OpenShift web console
- Build and deploy an application using s2i
- Inspect a BuildConfig and an ImageStream

# Verify the environment and command line tools
1. If a terminal is not already open, open a terminal window by using the menu in the editor: `Terminal > New Terminal`.
![New terminal](images/new-terminal.png)

2. Verify that `oc` CLI is installed.
```
oc version
```
You should see output similar to this, though the versions may be different:
```
Client Version: 4.5.0-202002280431-79259a8
Kubernetes Version: v1.16.2+45a4ac4
```

3. Change to your project folder.
```
cd /home/project
```

4. Clone the git repository that contains the artifacts needed for this lab, if it doesn't already exist.
```
[ ! -d 'cc201' ] && git clone https://gitlab.com/ibm/skills-network/courses/cc201.git
```

5. Change to the directory for this second lab.
```
cd cc201/labs/4_IntroOpenShift/
```

6. List the contents of this directory to see the artifacts for this lab.
```
ls
```

# Use the `oc` CLI
OpenShift projects are Kubernetes namespace with additional administrative functions. Therefore, projects also provide isolation within an OpenShift cluster. You already have access to one project in an OpenShift cluster, and `oc` is already set to target that cluster and project.

Let's look at some basic `kubectl` commands.

1. `kubectl` requires configuration so that it targets the appropriate cluster. Get cluster information with the following command:
```
kubectl config get-clusters
```

2. A `kubectl` context is a group of access parameters, including a cluster, a user, and a namespace. View your current context with the following command:
```
kubectl config get-contexts
```

3. List all the Pods in your namespace. If this is a new session for you, you will not see any Pods.
```
kubectl get pods
```

# Use the OpenShift web console
In addition to the CLI, OpenShift provides an intuitive web console. This is very useful and powerful, as you are able to deploy applications, view resources, monitor applications and view logs, and much more right in the console.

Let's open up the console and have a look around.

1. Open the OpenShift web console using the link at the top of the lab environment. It can take a few minutes to become available after opening the lab environment, so if you get an error, wait a minute and try again.
![Open OpenShift web console](images/open-web-console.png)

# Create a Pod with an imperative command
Now it's time to create your first Pod. This Pod will run the `hello-world` image you built and pushed to IBM Cloud Container Registry in the last lab. As explained in the videos for this module, you can create a Pod imperatively or declaratively. Let's do it imperatively first.

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

4. Run the `hello-world` image as a container in Kubernetes.
```
kubectl run hello-world --image us.icr.io/$MY_NAMESPACE/hello-world:1 --overrides='{ "spec": { "imagePullSecrets": [{"name": "icr"}] } }'
```
You will see a warning, but the Pod has been created for you. The `--overrides` option here enables us to specify the needed credentials to pull this image from IBM Cloud Container Registry. Note that this is an imperative command, as we told Kubernetes explicitly what to do: run `hello-world`.

5. List the Pods in your namespace.
```
kubectl get pods
```
Great, the previous command indeed created a Pod for us. You can see an auto-generated name was given to this Pod.

You can also specify the wide option for the output to get more details about the resource.
```
kubectl get pods -o wide
```

6. Note the Pod name from the previous step, and describe the Pod to get more details about it.
```
kubectl describe pod <pod_name>
```
Take a look at this output--there's a lot there. If you look closely, you'll notice that there is a ReplicaSet associated with this Pod. This is because the `kubectl run` command actually created a Deployment with one replica, which in turn created a ReplicaSet. At the end of the output, you'll also see events. These give some history for this resource. For example, you should see events that indicate that this Pod was scheduled, the image was pulled, and the container was started.

7. List the Deployments and ReplicaSets in your namespace to verify that one of each was created.
```
kubectl get deployments,replicasets
```

8. Delete the Deployment. This will also delete the ReplicaSet and the Pod.
```
kubectl delete deployment hello-world
```

9. List the Pods to verify that none exist.
```
kubectl get pods
```
The ReplicaSet and the Pod were deleted since the owning Deployment was deleted.

# Create a Pod with imperative object configuration
Imperative object configuration lets you create objects imperatively while using a configuration file. A configuration file, `hello-world-create.yaml`, is provided to you in this directory.

1. Use the Explorer to view and edit the configuration file. Click the Explorer icon (it looks like a sheet of paper) on the left side of the window, and then navigate to the directory for this lab: `cc201 > labs > Introduction to Kubernetes`. Click `hello-world-create.yaml` to view the configuration file.
![Imperative object configuration file in Explorer](images/imperative-obj-config-explorer.png)

2. Use the Explorer to edit `hello-world-apply.yaml`. You need to insert your namespace where it says `<my_namespace>`. Make sure to save the file when you're done.

3. Imperatively create a Pod using the provided configuration file.
```
kubectl create -f hello-world-create.yaml
```
Note that this is indeed imperative, as you explicitly told Kubernetes to *create* the resources defined in the file.

4. List the Pods in your namespace.
```
kubectl get pods
```
In this case, `kubectl` does not create a Deployment for us because the YAML file explicitly defined a Pod.

5. Delete the Pod.
```
kubectl delete pod hello-world
```
This command can take some time to run.

# Create a Pod with a declarative command
The previous two ways to create a Pod were imperative -- we explicitly told `kubectl` what to do. While the imperative commands are easy to understand and run, they are not ideal for a production environment. Let's look at declarative commands.

1. A sample `hello-world-apply.yaml` file is provided in this directory. Use the Explorer again to open this file. Notice the following:
- We are creating a Deployment (`kind: Deployment`).
- There will be three replica Pods for this Deployment (`replicas: 3`).
- The Pods should run the `hello-world` image (`- image: us.icr.io/<my_namespace>/hello-world:1`).
You can ignore the rest for now. We will get to a lot of those concepts in the next lab.

2. Use the Explorer to edit `hello-world-apply.yaml`. You need to insert your namespace where it says `<my_namespace>`. Make sure to save the file when you're done.

3. Use the `kubectl apply` command to set this configuration as the desired state in Kubernetes.
```
kubectl apply -f hello-world-apply.yaml
```

4. Get the Deployments to ensure that a Deployment was created.
```
kubectl get deployments
```

5. List the Pods to ensure that three replicas exist.
```
kubectl get pods
```
With declarative management, we did not tell Kubernetes which actions to perform. Instead, `kubectl` inferred that this Deployment needed to be created. If you delete a Pod now, a new one will be created in its place to maintain three replicas.

6. Note one of the Pod names from the previous step, and delete that Pod.
```
kubectl delete pod <pod_name>
```
This command can take some time to run.

7. List the Pods to see a new one being created.
```
kubectl get pods
```
If you do this quickly enough, you can see one Pod being terminated and another Pod being created.
```
NAME                    READY   STATUS              RESTARTS   AGE
hello-world-dd6b5d745-2jw5s   0/1     Terminating         0          35s
hello-world-dd6b5d745-f9xjk   1/1     Running             0          35s
hello-world-dd6b5d745-m89fc   0/1     ContainerCreating   0          8s
hello-world-dd6b5d745-qvs9t   1/1     Running             0          35s
```
Otherwise, the status of each will be the same, but the age of one Pod will be less than the others.
```
NAME                    READY   STATUS    RESTARTS   AGE
hello-world-dd6b5d745-f9xjk   1/1     Running   0          39s
hello-world-dd6b5d745-m89fc   1/1     Running   0          12s
hello-world-dd6b5d745-qvs9t   1/1     Running   0          39s
```

# Load balancing the application
Since there are three replicas of this application deployed in the cluster, Kubernetes will load balance requests across these three instances. Let's expose our application to the internet and see how Kubernetes load balances requests.

1. In order to access the application, we have to expose it to the internet using a Kubernetes Service.
```
kubectl expose deployment/hello-world --type=NodePort --port=8080 --name=hello-world --target-port=8080
```
This command creates what is called a NodePort Service. This will open up a port on the worker node to allow the application to be accessed using that port and the IP address of the node.

2. List Services in order to see that this service was created.
```
kubectl get services
```

3. Two things are needed to access this application: a worker node IP address and the correct port. To get a worker node IP, rerun the `get pods` command with the `wide` option and note any one of the node IP addresses (from the column entitled `NODE`):
```
kubectl get pods -o wide
```
Here is some sample output.
```
NAME                          READY   STATUS    RESTARTS   AGE   IP               NODE            NOMINATED NODE   READINESS GATES
hello-world-dd6b5d745-f9xjk   1/1     Running   0          7m    172.30.104.185   10.114.85.153   <none>           <none>
hello-world-dd6b5d745-m89fc   1/1     Running   0          7m    172.30.165.182   10.114.85.151   <none>           <none>
hello-world-dd6b5d745-qvs9t   1/1     Running   0          7m    172.30.69.68     10.114.85.172   <none>           <none>
```
Using this sample output, you could choose `10.114.85.153`, `10.114.85.151`, or `10.114.85.172` for the node IP address.

4. Export the node IP address as an environment variable. Make sure to substitute the copied IP address into this command.
```
export NODE_IP=<node_ip>
```
Using the sample output, one correct command would be `export NODE_IP=10.114.85.153`.

5. To get the port number, run the following command and note the port:
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
Using the sample output, the correct command would be `export NODE_PORT=31758`.

7. Ping the application to get a response.
```
curl $NODE_IP:$NODE_PORT
```

8. Notice that this output includes the Pod name. Run the command ten times and note the different Pod names in each line of output.
```
for i in `seq 10`; do curl $NODE_IP:$NODE_PORT; done
```
You should see more than one Pod name, and quite possibly all three Pod names, in the output. This is because Kubernetes load balances the requests across the three replicas, so each request could hit a different instance of our application.

9. Delete the Deployment and Service.
```
kubectl delete deployment hello-world service hello-world
```

Congratulations! You have completed the second lab of this course.
