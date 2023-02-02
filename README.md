# Kubernetes: Create Deployments Using YAML Files


Kubernetes is a container orchestration platform. It allows you to deploy, manage, and scale containerized applications through automation. In this article I will guide you on creating a deployment, adding a service to the deployment and updating the deployment using YAML configuration files. This method is the declarative management of Kubernetes objects, which is best for production and automation.


#### Prerequisites

* Preferred IDE, I will use Visual Studio Code.
* Docker Desktop with Kubernetes enabled, in Settings.

#### Deployment


In Kubernetes, the deployment is an object that you use to scale, roll out, and roll back the various versions of your application. We will create our deployment with a YAML file. This will create any number of identical pods that we specify. To get started, first open up VSCode and create a new directory to work from. Then create a new file. I will name mine nginx-deploy.yaml.


![image](https://user-images.githubusercontent.com/115881685/216232527-4f699e26-fc22-41ee-96a4-6411ef014797.png)



Here we will create our YAML file that is a template for the Kubernetes API to create the deployment. The YAML file provides the object spec which has 4 required fields.



* apiVersion- The version of the Kubernetes API you are using.
* kind- The kind of object you want to create.
* metadata- Data to identify the object, such as name.
* spec- The state you want for the object, reference kubectl explain services for more options.


For this step you can copy the nginx-deploy.yaml file below.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
        
        
This object tells Kubernetes API to deploy a single replicated pod with the nginx image on port 80. To use this file input the following command in a Terminal in VSCode. Indicating your file name.

```
kubectl apply -f nginx-deploy.yaml
```

The output will show that the deployment was created.


![image](https://user-images.githubusercontent.com/115881685/216233478-f93507de-0aad-45c5-a409-90fde78c97fd.png)



Now that it is created, let’s check it out! Input *kubectl get deployment* .


![image](https://user-images.githubusercontent.com/115881685/216233713-53d222ed-9e09-404d-8aee-9ceb3e5c6cd5.png)


This shows we have 1 replica running, which is what we indicated in our *YAML file*.


#### Service

Next, we will add a Service to our pod. A service is used to allow network access to the pods. We will once again create a YAML file to add the service. I named my file nginx-service.yaml. Copy and paste the gist below into VSCode.


```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30007
      ```
      
 

1
Note that the kind is Service and this will apply to all applications with the label nginx. The spec section includes a NodePort which is an open port on every node of your cluster that will route traffic on nodePort 30007. The NodePorts range is 30000–32768. Now once you save the file, run the command:



```
kubectl apply -f nginx-service.yaml
```


The output shows service created!



![image](https://user-images.githubusercontent.com/115881685/216234386-b26dcfc0-a8a7-46cd-82a6-5ca6fcf6484f.png)



To check out the service type in the command **kubectl get service**. The LoadBalancer will be added to the list that will contain an ExternalIP that will be in pending state.



![image](https://user-images.githubusercontent.com/115881685/216234795-738ed258-9750-4fe2-956f-4c4a33f9559b.png)



To expose, the LoadBalancer url, run **minikube service nginx-service** command, which automatically opens the nginix-service in a browser.



![image](https://user-images.githubusercontent.com/115881685/216235977-8d9193cf-14e1-496a-99f7-0465c62cbe03.png)



![image](https://user-images.githubusercontent.com/115881685/216236063-7928c1ed-60c1-41ea-a356-c227ac396b85.png)



#### Update


Finally, I will update the deployment to include 4 replicas. To do this we only need to update the **YAML file nginx-deploy.yaml**. First input the following command:




```
kubectl get pods -w
```


The -w enables us to watch as we make updates to the cluster. So for now you should only see one pod running. Next I like to split the terminal to better visualize the changes happening. In your terminal right click on the terminal name and select Split Terminal.



![image](https://user-images.githubusercontent.com/115881685/216236446-534b4cdc-d159-4014-9147-5c86cb826b03.png)



Next update your deployment YAML file to indicate 4 replicas.




![image](https://user-images.githubusercontent.com/115881685/216236730-8a285539-deb7-4018-84d8-f4edc730adb4.png)




Save it and run the command again.



```
kubectl apply -f nginx-deploy.yaml
```



You will see as the changes take place that 3 more replicas will be added. Also the output will show that the deployment was configured.




![image](https://user-images.githubusercontent.com/115881685/216237155-b6289411-f7d2-4137-b913-c2b31e63cd31.png)




```
Enter **kubectl get deployment** to check the deployment information.



![image](https://user-images.githubusercontent.com/115881685/216237371-629a0bc2-c544-426a-9c2a-7c5b01cdce05.png)



4 pods running! Just as anticipated.




Another thing Kubernetes does is auto-scaling. I will demonstrate this by deleting one of my pods and seeing if it scales out. First type in **kubectl get pods**. Copy one of the pods name.




![image](https://user-images.githubusercontent.com/115881685/216237545-8a526acc-682e-4add-9b9b-d0a19f1452fd.png)



Then run the following command to delete the pod. Be sure to put in your pods name.



```
kubectl delete pod/nginx-deployment-7fb96c846b-57x2d
```


It will quickly spin up a new pod to replace it.



![image](https://user-images.githubusercontent.com/115881685/216238139-62684697-b752-4220-84da-8e88a48a5b8e.png)



You can check the deployment again to ensure you still have 4 running.




![image](https://user-images.githubusercontent.com/115881685/216238252-6ec08ffa-8708-4c34-b17f-cddb245ec372.png)




And we do!



To clean up:



```
kubectl delete -f nginx-deploy.yaml
kubectl delete -f nginx-service.yaml
```



![image](https://user-images.githubusercontent.com/115881685/216238694-7f07743c-142a-4e9c-a354-6dfd63fabeb7.png)
![image](https://user-images.githubusercontent.com/115881685/216238756-cb434ec1-2c8e-4bde-83a9-ddf739f1289d.png)





Now you could combine the service and deployment files into one file and apply it. Or you could apply the whole directory. This article just shows a beginners guide to understanding the YAML configuration files.
































        
        
        








