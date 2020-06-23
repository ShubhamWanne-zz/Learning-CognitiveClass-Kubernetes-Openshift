# Introduction to Kubernetes

## Objectives

In this lab, you will:
- work with the `kubectl` CLI.
- create a Kubernetes Pod.
- create a Kubernetes Deployment.

## Prerequisites

The complete environment needed for this lab is provided to you at this link <tbd>.

# Ensure environment is installed
1. Open the terminal in the provided environment by using the menu in the editor: `Terminal > New Terminal`.

   ![terminal](images/terminal.png)

1. Make sure the environment is configured by running the following commands in the terminal:
   - Verify `docker` is installed.
        ```
        theia@theiadocker-ulidder:/home/project$ docker --version
        >> Docker version 18.09.7, build 2d0083d
        ```
   - Verify `ibmcloud` CLI is installed.
        ```
        theia@theiadocker-ulidder:/home/project$ ibmcloud --version
        >> ibmcloud version 1.0.0+908f90a-2020-03-30T06:37:22+00:00

2. Make sure `kubectl` is installed by running the following command. You should see output that looks similar to this, though the versions might be different.
   ```
   $ kubectl version
   
      Client Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.2", GitCommit:"59603c6e503c87169aea6106f57b9f242f64df89", G
      itTreeState:"clean", BuildDate:"2020-01-18T23:30:10Z", GoVersion:"go1.13.5", Compiler:"gc", Platform:"linux/amd64"}
      Server Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.10+IKS", GitCommit:"a0052bd119c067cf48e8a19f0ab7d5a5e2ca0a1
      8", GitTreeState:"clean", BuildDate:"2020-05-20T20:48:06Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
   ```

# Work with the `kubectl` CLI
The `kubectl` CLI is already installed for you in this environment. In addition, you already have access to one namespace in a Kubernetes cluster, and `kubectl` is already set up to target that cluster and namespace.

Let's look at some basic `kubectl` commands.

1. Get cluster information with the following command.
   ```
   $ kubectl config get-clusters
   
      NAME
      labs-user-sandbox-prod-tor01/bp1hokjw0e5bk715r7f0
   ```
2. List all the Pods in your namespace. If this is a new session for you, you will not see any Pods.
      ```
      $ kubectl get pods
      
         No resources found in sn-labs-upkaribmwats namespace.
      ```

# Create a Pod
Let's create our first Pod. This Pod will run the `nginx` image from Docker Hub. As explained in the videos for this module, you can create a Pod imperatively or declaratively.

## Create a Pod with an imperative command
1. Run the `nginx` image as a container in Kubernetes.
   ```
      $ kubectl run nginx --image nginx

         kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.

         deployment.apps/nginx created
   ```
   You will see a warning, but the Pod has been created for you. Note that this is an imerpative command as we told Kubernetes explicitly what to do: run `nginx`.
1. List the Pods in your namespace.
   ```
   $ kubectl get pods
      NAME                     READY   STATUS    RESTARTS   AGE
      nginx-6db489d4b7-2nzd8   1/1     Running   0          2m55s
   ```
   Great, the previous command indeed created a Pod for us. It is named `nginx-6db489d4b7-2nzd8`. Your Pod will most likely have a different name.

   You can also specify the wide option for the output to get more details about the resource.
   ```
   $ kubectl get pods -o wide
      NAME                     READY   STATUS    RESTARTS   AGE     IP             NODE            NOMINATED NODE   READINESS GATES
      nginx-6db489d4b7-2nzd8   1/1     Running   0          9m12s   172.30.69.74   10.114.85.172   <none>           <none>
   ```
1. Delete the Pod.
   ```
   $ kubectl delete pod <opd_name>
   
      pod "nginx-6db489d4b7-2nzd8" deleted
   ```
1. Get the Pods to verify that the Pod was deleted.
   ```
      $ kubectl get pods

         NAME                     READY   STATUS    RESTARTS   AGE
         nginx-6db489d4b7-rstbf   1/1     Running   0          77s
   ```
   Wait a second! We deleted the Pod in the previous step, so why is it still there? The reason is that the `kubectl run` command actually created a `Deployment` with one replica. Therefore, Kubernetes automatically created a Pod for us when we deleted the original one. You will see the pod name is different now.
1. Describe the Pod to get more details about it.
   ```
   $ kubectl describe pod nginx-6db489d4b7-rstbf

   Name:         nginx-6db489d4b7-rstbf
   Namespace:    sn-labs-upkaribmwats
   Priority:     0
   Node:         10.114.85.172/10.114.85.172
   Start Time:   Thu, 18 Jun 2020 23:12:25 +0000
   Labels:       pod-template-hash=6db489d4b7
               run=nginx
   Annotations:  kubernetes.io/limit-ranger:
                  LimitRanger plugin set: cpu, ephemeral-storage, memory request for container nginx; cpu, ephemeral-storage, memor
   y limit for container ngi...
               kubernetes.io/psp: ibm-privileged-psp
   Status:       Running
   IP:           172.30.69.74
   IPs:
   IP:           172.30.69.74
   Controlled By:  ReplicaSet/nginx-6db489d4b7
   Containers:
   nginx:
      Container ID:   containerd://778b66a518452a8eb7209ab42ae2f6af326a8f686f780199368764700c38d6e8
      Image:          nginx
      Image ID:       docker.io/library/nginx@sha256:21f32f6c08406306d822a0e6e8b7dc81f53f336570e852e25fbe1e3e3d0d0133
      Port:           <none>
      Host Port:      <none>
      State:          Running
         Started:      Thu, 18 Jun 2020 23:12:27 +0000
      Ready:          True
      Restart Count:  0
      Limits:
         cpu:                500m
         ephemeral-storage:  3Gi
         memory:             512Mi
      Requests:
         cpu:                200m
         ephemeral-storage:  512Mi
         memory:             128Mi
      Environment:          <none>
      Mounts:
         /var/run/secrets/kubernetes.io/serviceaccount from default-token-n6v4s (ro)
   ...
   ...
   ```
   As you can see, the Pod has a ReplicaSet associated with it.
   
1. List the Deployments in your namespace to verify that a Deployment was created.
   ```
      $ kubectl get deploy
         NAME    READY   UP-TO-DATE   AVAILABLE   AGE
         nginx   1/1     1            1           32m
   ```
1. Delete the Deployment.
   ```
      $ kubectl delete deploy nginx
         deployment.apps "nginx" deleted  
   ```
1. List the Pods to verify that none exist.
   ```
      $ kubectl get pods
         No resources found in sn-labs-upkaribmwats namespace.
   ```
   The ReplicaSet and the Pod were deleted since the owning Deloyment was deleted.

## Create a Pod with imperative object configuration
Imperative object configuration lets you create objects imperatively while using a configuration file. A configuration file `nginx-create.yaml` is provided to you in this directory.

WE SHOULD HAVE THEM INSPECT THE FILE HERE. I need to view the environment to know how to write this best.

1. Imperatively create a Pod using the provided configuration file.
   ```
      $ kubectl create -f nginx-create.yaml 
         pod/nginx created
   ```
   Note that this is indeed imperative as you explicitly told Kubernetes to *create* the resources defined in the file.
1. List the Pods in your namespace.
   ```
      $ kubectl get pods
         NAME    READY   STATUS    RESTARTS   AGE
         nginx   1/1     Running   0          5s
   ```
   In this case, `kubectl` does not create a Deployment for us, because the YAML file explicitly defined a Pod.
1. Delete the Pod.
   ```
      $ kubectl delete pod nginx
         pod "nginx" deleted
   ```

## Create a Pod with a declarative command
The previous two ways to create a pod were imperative—we explicitly told `kubectl` what to do. While the imperative commands are easy to understand and execute, they are not ideal for a production environment. Let's look at declarative commands.

1. A sample nginx-apply.yaml file is provided in this directory. Open that file in the editor. Notice the following:
   1. We are creating a Deployment.
   ```
      kind: Deployment
   ```
   1. There will be three replica Pods for this Deployment.
   ```
      replicas: 3
   ```
   1. The Pods should run the `nginx` image.
   ```
      containers:
      - image: nginx:latest
        imagePullPolicy: Always
        name: nginx
   ```
   You can ignore the rest for now. We will get to a lot of those concepts in the next lab.
1. Use the `kubectl apply` command to set this configuration as the desired state in Kubernetes.
   ```
   $ kubectl apply -f nginx-apply.yaml
      deployment.apps/nginx created
   ```
1. Get the Deployments to ensure that one was created.
   ```
      $ kubectl get deploy
         NAME    READY   UP-TO-DATE   AVAILABLE   AGE
         nginx   3/3     3            3           18s
   ```
1. List the Pods to ensure that three replicas exist.
   ```
      $ kubectl get pods
         
         NAME                    READY   STATUS    RESTARTS   AGE
         nginx-dd6b5d745-8g4ck   1/1     Running   0          17s
         nginx-dd6b5d745-frks2   1/1     Running   0          17s
         nginx-dd6b5d745-zrtb6   1/1     Running   0          17s
   ```
With declarative management, we did not tell Kubernetes what actions needed to be performed. Instead, `kubectl` inferred that this Deployment needed to be created, so it created it. If you delete a Pod now, a new one will be created in its place to maintain three replicas.
1. Delete one of the three Pods.
   ```
      $ kubectl delete pod <pod_name>
         
         pod "nginx-dd6b5d745-8g4ck" deleted

   ```
1. List the Pods to see a new one being created.
   ```
   $ kubectl get pods
   ```
   If you do this quickly enough, you can see the Pod being created.
   ```
      NAME                    READY   STATUS              RESTARTS   AGE
      nginx-dd6b5d745-frks2   1/1     Running             0          3m7s
      nginx-dd6b5d745-xqmbf   0/1     ContainerCreating   0          11s
      nginx-dd6b5d745-zrtb6   1/1     Running 
   ```
   Otherwise, the statuses will all be the same, but the age of one Pod will be smaller than the others.
   ```
      $ kubectl get pods

         NAME                    READY   STATUS    RESTARTS   AGE
         nginx-dd6b5d745-frks2   1/1     Running   0          3m50s
         nginx-dd6b5d745-xqmbf   1/1     Running   0          54s
         nginx-dd6b5d745-zrtb6   1/1     Running   0 
   ```

Congratulations! You have completed the second lab of this course.