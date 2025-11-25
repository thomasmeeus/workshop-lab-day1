# OpenShift 4 Introduction lab

## 1. Welcome

This lab provides a quick tour of the OpenShift console to help you get familiar with the user interface. If you are already familiar with the basics of OpenShift, this will be easy in that we are simply ensuring you can login and create a project.

### Accessing OpenShift

OpenShift provides a web console that allows you to perform various tasks via a web browser.

Use your browser to navigate to the URI your OpenShift cluster and login with your personal username & password.

![img](images/ocp-login.png)

Once logged in you should see your available projects - or you will be provided with an informational box that “No projects exist”

![img](images/ocp-dev-view.png)

So this is what an empty project looks like
First let’s create a new project to do our workshop work in. Use `lab1-$USER` as your projectname

Click on the “Project: all projects” button and select “Create Project” from the drop down menu

![img](images/ocp-dev-create-project-terminal.gif)

### Let’s launch a terminal.

Top-right on the webinterface there's a dropdown which contains the option to 'copy login command'. 
You can be prompted to authenticate again. You can use the generated string to authenticate from the CLI.

Open a terminal on your local workstation and execute the `oc login $OPENSHIFT_URI` command.

You will be prompted for your credentials. 
Once logged in, you can execute `oc get projects` to get an overview of all projects. The project created earlier can be selected via 

```
oc project lab1-$USER
```

Inspect the yaml file of the project via

```
oc get project lab1-$USER -o yaml
```

### Inspect the openshift specific annotations:
 * description
 * displayname
 * requester
 * uid-range

## 2. Bring your own OCI image

It’s easy to get started with OpenShift whether you’re using our app templates or bringing your existing assets. In this quick lab we will deploy an application using an exisiting container image. OpenShift will create an image stream for the image as well as deploy and manage containers based on that image.

### Let’s point OpenShift to an existing container image

Launch a pre-existing container from a container registry.

```
oc new-app sonatype/nexus:oss --as-deployment-config=true

```

The output should show something *similar* to below:

```
--> Found container image 8027e6d (2 months old) from Docker Hub for "sonatype/nexus:oss"         
                                                                                                  
    Red Hat Universal Base Image 7                                                                
    ------------------------------                                                                
    The Universal Base Image is designed and engineered to be the base layer for all of your conta
inerized applications, middleware and utilities. This base image is freely redistributable, but Re
d Hat only supports Red Hat technologies through subscriptions for Red Hat products. This image is
 maintained by Red Hat and updated regularly.                                                     
                                                                                                  
    Tags: base rhel7                                                                              
                                                                                                  
    * An image stream tag will be created as "nexus:oss" that will track this image               
    * This image will be deployed in deployment config "nexus"                                    
    * Port 8081/tcp will be load balanced by service "nexus"                                      
      * Other containers can access this service through the hostname "nexus"                     
    * This image declares volumes and will default to use non-persistent, host-local storage.     
      You can add persistent volumes later by running 'oc set volume dc/nexus --add ...'          
                                                                                                  
--> Creating resources ...                                                                        
    imagestream.image.openshift.io "nexus" created                                                
    deploymentconfig.apps.openshift.io "nexus" created                                            
    service "nexus" created                                                                       
--> Success                                                                                       
    Application is not exposed. You can expose services to the outside world by executing one or m
ore of the commands below:                                                                        
     'oc expose svc/nexus'                                                                        
    Run 'oc status' to view your app.  

```

Review the output and notice that OpenShift suggest some next steps to be done after your deployment. 

Now, let's create a route, so that you can get to the app:

```
$ oc expose service/nexus --path=/nexus
```

## 2.1 Reviewing container details

Now that we have a running container, we can browse our project details with the command line

Try typing the following to see what is available to ‘get’:

```
$ oc project lab1-$USER
$ oc get all
```

Now let’s look at what our image stream has in it:

```
$ oc get is
```

```
$ oc describe is/nexus
```

An image stream can be used to automatically perform an action, such as updating a deployment, when a new image, in our case a new version of the nexus image, is created.

The app is running in a pod, let’s look at that:

```
$ oc describe pods
```

Follow the application logs to make sure that it starts correctly:

```
$ oc logs dc/nexus -f
```

### Test out the nexus webapp

Fetch the route of our newly deployed Nexus instance and visit the webapp:

```bash
echo "http://$(oc get route nexus -o jsonpath='{.spec.host}{.spec.path}')"
````

Note the use of jsonpath expressions to fetch the proper data's from the OpenShift API.

Browse to your webapp and verify that the Nexus application loads fine. 

![img](images/ocp-nexus-app2.png)

### Troubleshooting

Append '/nexus' if you get the following when attempting to browse the route. This means you probably skipped one of the previous steps

![img](images/ocp-nexus-app.png)


### Let’s clean this up

Let’s clean up all this to get ready for the next lab:

```
$ oc project lab1-$USER
$ oc delete all --selector app=nexus
```

### Summary

In this lab, you’ve deployed an example container image into a pod running in OpenShift. You exposed a route for clients to access that service via their web browsers. And you learned how to get and describe resources using the command line and the web console. Hopefully, this basic lab also helped to get you familiar with using the CLI and navigating within the web console.

## 4. Deploying an the Metro Map App

Deploy the Metro Map application which is hosted on the Quay.io registry:

```
$ oc project lab1-$USER
$ oc new-app --image quay.io/thomasmeeus/metro-map:latest --name dc-metro-map
$ oc expose service dc-metro-map
```

### Check out the events

We can see the details of Openshift did to deploy this app:

Goto the terminal and type the following:

```
$ oc get events
```

You can also view the pod logs via 

```
$ oc logs pod/dc-metro-map-xxxx
```

The console will print out the full log for your app. Note, you could pipe this to `more` or `less` for easier viewing in the CLI.

Both outputs can also be viewed in the webinterface, try to discover their location!

### See the app in action
Let’s see this app in action!


Goto the terminal and type the following (ensure you are in the right project):
```
$ oc get routes
```

Copy the HOST/PORT and paste into your favorite web browser:

![img](images/dc-metro-map-app.png)

Clicking the checkboxes will toggle on/off the individual metro stations on each colored line. A numbered icon indicates there is more than one metro station in that area and they have been consolidated - click the number or zoom in to see more.

### Summary
In this lab we deployed a sample application using source to image. This process built our code and wrapped that in a docker image. It then deployed the image into our OpenShift platform in a pod and exposed a route to allow outside web traffic to access our application. In the next lab we will look at some details of this app’s deployment and make some changes to see how OpenShift can help to automate our development processes. 

## 5. Developing and Managing Your Application

### Developing and managing an application in OpenShift

In this lab we will explore some of the common activities undertaken by developers working in OpenShift. You will become familiar with how to use environment variables, secrets, build configurations, and more. Let’s look at some of the basic things a developer might care about for a deployed app.

### Setup

From the previous lab you should have the DC Metro Maps web app running in OpenShift.

### See the app in action and inspect some details

Unlike in previous versions of OpenShift, there is no more ambiguity or confusion about where the app came from. OpenShift provides traceability for your running deployment, back to the container image, and the registry that it came from. Additionally, images built by OpenShift are traceable back to the exact branch and commit. Let’s take a look at that!

### Goto the terminal and type the following:

```
$ oc status
```

This is going to show the status of your current project. In this case it will show the dc-metro-map service (svc) with a nested deployment config(also called a “DC”) along with some more info that you can ignore for now.

A deployment in OpenShift is a replication controller based on a user defined template called a deployment configuration
The dc provides us details we care about to see where our application image comes from, so let’s check it out in more detail.

Type the following to find out more about our dc:

```
$ oc describe deployment/dc-metro-map
```

Notice under the template section it lists the containers it wants to deploy along with the path to the container image.

There are a few other ways you could get to this information. A similar output can be seen when describing the pod's content:

`oc describe pod`

We can verify the source of the image, its registry and the tag that got deployed. We also see on which port the application is listening on. Take note of the 'conditions' which display a health overview of your pod.

Copy the registry image url in your webbrowser and verify the details of the image in the Quay web interface. Verify the tags, history and security status of the provided image. 

## 6. Pod Logs

Let’s inspect the log for a running pod - in particular let’s see the web application’s logs.

Goto the terminal and type the following:

```
$ oc get pods
```

This is going to show basic details for all pods in this project (including the builders). Let’s look at the log for the pod running our application. Look for the POD NAME that that is “Running” you will use it below.

 Goto the terminal and type the following (replacing the POD ID with your pod's ID):

```
$ oc logs [POD NAME]
```

You will see in the output details of your app starting up and any status messages it has reported since it started. Visist the application a couple of times and you'll see new logs being outputted to your console.

## 7. How about we set some environment variables?

Whether it’s a database name or a configuration variable, most applications make use of environment variables. It’s best not to bake these into your containers because they do change and you don’t want to rebuild an image just to change an environment variable. Good news! You don’t have to. OpenShift let’s you specify environment variables in your deployment configuration and they get passed along through the pod to the container. Let’s try doing that.

Let’s have a little fun. The app has some easter eggs that get triggered when certain environment variables are set to ’true'.

Goto the terminal and type the following:

```
$ oc set env deployment/dc-metro-map -e BEERME=true
$ oc get pods -w
```

Due to the deployment config strategy being set to “Rolling” and the “ConfigChange” trigger being set, OpenShift auto deployed a new pod as soon as you updated the environment variable. If you were quick enough, you might have seen this happening, with the “oc get pods -w” command

Type Ctrl+C to stop watching the pods

With the new environment variables set the app should look like this in your web browser (with beers instead of buses ):

![img](images/ocp-lab-devman-beerme.png)

## 8. Getting into a pod

There are situations when you might want to jump into a running pod, and OpenShift lets you do that pretty easily. We set some environment variables, in this lab, so let’s jump onto our pod to inspect them.

Goto the terminal and type the following:

```
$ oc get pods
```

Find the pod name for your Running pod

```
$ oc rsh [POD NAME]
```

You are now interactively attached to the container in your pod. Let’s look for the environment variables we set:

```
$ env | grep BEER
```

That should return the BEERME=true matching the value that we set in the deployment config.

```
$ exit
```


## 10. Replication & Recovery

Things will go wrong with your software, or your hardware, or from something completely out of your control. But, we can plan for such failures, thus minimizing their impact. OpenShift supports this via the replication and recovery functionality.

### Replication

Let’s walk through a simple example of how the replication controller can keep your deployment at a desired state. Assuming you still have the dc-metro-map project running we can manually scale up our replicas to handle increased user load.

Goto the terminal and try the following:

```
$ oc scale --replicas=4 deployment/dc-metro-map
```

Check out the new pods:

```
$ oc get pods
```

Notice that you now have 4 unique pods available to inspect. If you want go ahead and inspect them, using ‘oc describe pod/POD NAME’. You can see that each has its own IP address and logs.

So you’ve told OpenShift that you’d like to maintain 4 running, load-balanced, instances of our web app.

### Recovery

Okay, now that we have a slightly more interesting replication state, we can test a service outages scenario. In this scenario, the dc-metro-map replication controller will ensure that other pods are created to replace those that become unhealthy. Let’s forcibly inflict an issue and see how OpenShift responds.

Choose a random pod and delete it:

```
$ oc get pods
$ oc delete pod/PODNAME
$ oc get pods -w
```

If you’re fast enough you’ll see the pod you deleted go “Terminating” and you’ll also see a new pod immediately get created and transition from “Pending” to “Running”. If you weren’t fast enough you can see that your old pod is gone and a new pod is in the list with an age of only a few seconds.

You can see the more details about your deployment configuration with:

```
$ oc describe deployment/dc-metro-map
```

Inspect the 'Replicas:' & 'RollingUpdateStrategy:' output.

### Application Health

In addition to the health of your application’s pods, OpenShift will watch the containers inside those pods. Let’s forcibly inflict some issues and see how OpenShift responds.

Choose a running pod and shell into it:

```
$ oc get pods
$ oc rsh <PODNAME>
```

You are now executing a bash shell running in the container of the pod. Let’s kill our webapp and see what happens.

If we had multiple containers in the pod we could use "-c CONTAINER_NAME" to select the right one
Choose a running pod and shell into its container:

```
$ pkill -9 node
```

This will kick you out off the container with an error like “Error executing command in container”

Do it again - shell in and execute the same command to kill node
Watch for the container restart

```
$ oc get pods -w
```

If a container dies multiple times quickly, OpenShift is going to put the pod in a CrashBackOff state. This ensures the system doesn’t waste resources trying to restart containers that are continuously crashing.


### Summary

In this lab we learned about replication controllers and how they can be used to scale your applications and services. We also tried to break a few things and saw how OpenShift responded to heal the system and keep it running.


## 11. Labels
 
This is a pretty simple lab, we are going to explore labels. You can use labels to organize, group, or select API objects.

For example, pods are “tagged” with labels, and then services use label selectors to identify the pods they proxy to. This makes it possible for services to reference groups of pods, even treating pods with potentially different docker containers as related entities.

### Labels on a pod

In a previous lab we added our web app using a S2I template. When we did that, OpenShift labeled our objects for us. Let’s look at the labels on our running pod.

Goto the terminal and try the following:

```
$ oc get pods
$ oc describe pod/<POD NAME> | grep Labels: --context=4
Namespace:    demo-1
Priority:     0
Node:         ip-10-0-132-38.us-east-2.compute.internal/10.0.132.38
Start Time:   Tue, 14 Apr 2020 17:41:58 +0000
Labels:       app=dc-metro-map
              deploymentconfig=dc-metro-map
              pod-template-hash=7bc46bf89d
Annotations:  k8s.v1.cni.cncf.io/networks-status:
                [{
```

You can see the Labels automatically added contain the app, deployment, and deploymentconfig. Let’s add a new label to this pod.

### Add a label

```
$ oc label pod/<POD NAME> testdate=4.14.2020 testedby=mylastname
```

Display the labels again & verify the label was added:

```
$ oc describe pod/<POD NAME> | grep Labels: --context=4
Namespace:    demo-1
Priority:     0
Node:         ip-10-0-132-38.us-east-2.compute.internal/10.0.132.38
Start Time:   Tue, 14 Apr 2020 17:41:58 +0000
Labels:       app=dc-metro-map
              deploymentconfig=dc-metro-map
              pod-template-hash=7bc46bf89d
              testdate=4.14.2020
              testedby=mylastname
```

### Cleanup

Cleanup your project & all resources when finished:

```
oc delete project lab1-$USER
```

This workshop is based on https://redhatgov.io/workshops/openshift_4_101/. Other interesting course are:

* https://developers.redhat.com/learn/openshift/perform-place-kubernetes-updates-bluegreen-deployment

* https://developers.redhat.com/learn/openshift/store-persistent-data-red-hat-openshift-using-pvcs
