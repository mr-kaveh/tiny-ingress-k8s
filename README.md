# tiny-ingress-k8s
A simple NGINX ingress test based on [k8s docs reference](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/)


### Create a minikube cluster[](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/#create-a-minikube-cluster)

If you haven't already set up a cluster locally, run  `minikube start`  to create a cluster.

## Enable the Ingress controller[](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/#enable-the-ingress-controller)

1.  To enable the NGINX Ingress controller, run the following command:
    
    ```shell
    minikube addons enable ingress
    
    ```
    
2.  Verify that the NGINX Ingress controller is running
    
    ```shell
    kubectl get pods -n ingress-nginx
    
    ```
    
    **Note:**  It can take up to a minute before you see these pods running OK.
    
    The output is similar to:
    
    ```none
    NAME                                        READY   STATUS      RESTARTS    AGE
    ingress-nginx-admission-create-g9g49        0/1     Completed   0          11m
    ingress-nginx-admission-patch-rqp78         0/1     Completed   1          11m
    ingress-nginx-controller-59b45fb494-26npt   1/1     Running     0          11m
    
    ```
    

## Deploy a hello, world app[](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/#deploy-a-hello-world-app)

1.  Create a Deployment using the following command:
    
    ```shell
    kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0
    
    ```
    
    The output should be:
    
    ```none
    deployment.apps/web created
    
    ```
    
    Verify that the Deployment is in a Ready state:
    
    ```shell
    kubectl get deployment web 
    
    ```
    
    The output should be similar to:
    
    ```none
    NAME   READY   UP-TO-DATE   AVAILABLE   AGE
    web    1/1     1            1           53s
    
    ```
    
2.  Expose the Deployment:
    
    ```shell
    kubectl expose deployment web --type=NodePort --port=8080
    
    ```
    
    The output should be:
    
    ```none
    service/web exposed
    
    ```
    
3.  Verify the Service is created and is available on a node port:
    
    ```shell
    kubectl get service web
    
    ```
    
    The output is similar to:
    
    ```none
    NAME      TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
    web       NodePort   10.104.133.249   <none>        8080:31637/TCP   12m
    
    ```
    
4.  Visit the Service via NodePort, using the  [`minikube service`](https://minikube.sigs.k8s.io/docs/handbook/accessing/#using-minikube-service-with-tunnel)  command. Follow the instructions for your platform:
    
    -   [Linux](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/#minikube-service-0)
    -   [MacOS](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/#minikube-service-1)
    
    ```shell
    minikube service web --url
    
    ```
    
    The output is similar to:
    
    ```none
    http://172.17.0.15:31637
    
    ```
    
    Invoke the URL obtained in the output of the previous step:
    
    ```shell
    curl http://172.17.0.15:31637 
    
    ```
    
      
    The output is similar to:
    
    ```none
    Hello, world!
    Version: 1.0.0
    Hostname: web-55b8c6998d-8k564
    
    ```
    
    You can now access the sample application via the Minikube IP address and NodePort. The next step lets you access the application using the Ingress resource.
    

## Create an Ingress[](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/#create-an-ingress)

The following manifest defines an Ingress that sends traffic to your Service via  `hello-world.info`.

1.  Create  `example-ingress.yaml`  from the following file:
    
    [`service/networking/example-ingress.yaml`](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/service/networking/example-ingress.yaml) ![](https://kubernetes.io/images/copycode.svg "Copy service/networking/example-ingress.yaml to clipboard")
    
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: example-ingress
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /$1
    spec:
      rules:
        - host: hello-world.info
          http:
            paths:
              - path: /
                pathType: Prefix
                backend:
                  service:
                    name: web
                    port:
                      number: 8080
    ```
    
2.  Create the Ingress object by running the following command:
    
    ```shell
    kubectl apply -f https://k8s.io/examples/service/networking/example-ingress.yaml
    
    ```
    
    The output should be:
    
    ```none
    ingress.networking.k8s.io/example-ingress created
    
    ```
    
3.  Verify the IP address is set:
    
    ```shell
    kubectl get ingress
    
    ```
    
    **Note:**  This can take a couple of minutes.
    
    You should see an IPv4 address in the  `ADDRESS`  column; for example:
    
    ```none
    NAME              CLASS    HOSTS              ADDRESS        PORTS   AGE
    example-ingress   <none>   hello-world.info   172.17.0.15    80      38s
    
    ```
    
4.  Verify that the Ingress controller is directing traffic, by following the instructions for your platform:
    
    **Note:**  The network is limited if using the Docker driver on MacOS (Darwin) and the Node IP is not reachable directly. To get ingress to work you’ll need to open a new terminal and run  `minikube tunnel`.  
    `sudo`  permission is required for it, so provide the password when prompted.
    
    -   [Linux](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/#ingress-0)
    -   [MacOS](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/#ingress-1)
    
    ```shell
    curl --resolve "hello-world.info:80:$( minikube ip )" -i http://hello-world.info
    
    ```
    
      
    You should see:
    
    ```none
    Hello, world!
    Version: 1.0.0
    Hostname: web-55b8c6998d-8k564
    
    ```
    
5.  Optionally, you can also visit  `hello-world.info`  from your browser.
    
    Add a line to the bottom of the  `/etc/hosts`  file on your computer (you will need administrator access):
    
    -   [Linux](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/#hosts-0)
    -   [MacOS](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/#hosts-1)
    
    Look up the external IP address as reported by minikube
    
    ```none
      minikube ip 
    
    ```
    
      
    
    ```none
      172.17.0.15 hello-world.info
    
    ```
    
    **Note:**  Change the IP address to match the output from  `minikube ip`.
    
      
    
    After you make this change, your web browser sends requests for  `hello-world.info`  URLs to Minikube.
    

## Create a second Deployment[](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/#create-a-second-deployment)

1.  Create another Deployment using the following command:
    
    ```shell
    kubectl create deployment web2 --image=gcr.io/google-samples/hello-app:2.0
    
    ```
    
    The output should be:
    
    ```none
    deployment.apps/web2 created
    
    ```
    
    Verify that the Deployment is in a Ready state:
    
    ```shell
    kubectl get deployment web2 
    
    ```
    
    The output should be similar to:
    
    ```none
    NAME   READY   UP-TO-DATE   AVAILABLE   AGE
    web2   1/1     1            1           16s
    
    ```
    
2.  Expose the second Deployment:
    
    ```shell
    kubectl expose deployment web2 --port=8080 --type=NodePort
    
    ```
    
    The output should be:
    
    ```none
    service/web2 exposed
    
    ```
    

## Edit the existing Ingress[](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/#edit-ingress)

1.  Edit the existing  `example-ingress.yaml`  manifest, and add the following lines at the end:
    
    ```yaml
    - path: /v2
      pathType: Prefix
      backend:
        service:
          name: web2
          port:
            number: 8080
    
    ```
    
2.  Apply the changes:
    
    ```shell
    kubectl apply -f example-ingress.yaml
    
    ```
    
    You should see:
    
    ```none
    ingress.networking/example-ingress configured
    
    ```
    

## Test your Ingress[](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/#test-your-ingress)

1.  Access the 1st version of the Hello World app.
    
    -   [Linux](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/#ingress2-v1-0)
    -   [MacOS](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/#ingress2-v1-1)
    
    ```shell
    curl --resolve "hello-world.info:80:$( minikube ip )" -i http://hello-world.info
    
    ```
    
      
    
    The output is similar to:
    
    ```none
    Hello, world!
    Version: 1.0.0
    Hostname: web-55b8c6998d-8k564
    
    ```
    
2.  Access the 2nd version of the Hello World app.
    
    -   [Linux](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/#ingress2-v2-0)
    -   [MacOS](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/#ingress2-v2-1)
    
    ```shell
    curl --resolve "hello-world.info:80:$( minikube ip )" -i http://hello-world.info/v2
    
    ```
    
    The output is similar to:
    
    ```none
    Hello, world!
    Version: 2.0.0
    Hostname: web2-75cd47646f-t8cjk
    ```
