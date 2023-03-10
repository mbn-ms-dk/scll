# scll
Lunch and Learn demo project

Showing Application Insights and Application Insights Kubernetes

## Get Started

This is a quick guide.

### Prerequisite

* [Docker Containers](https://www.docker.com/)
* [Kubernetes](https://kubernetes.io/)

> ⚠️ For RBAC-enabled Kubernetes, make sure [permissions are configured](./k8s/configure-rbac-permissions.md).

## Prepare image
1. Get the **dockerfile** ready. This is already created in this project
2. Build the image
```shell
docker build -f .\ApiService\Dockerfile --force-rm -t "aik8s-api:v5" --label "sclldemo"
docker run -d -p 8087:80 --name sclldemo aik8s-api:v5  
```
3. Check the logs
```shell
docker logs sclldemo
```
4. Clean up
```shell
docker container rm sclldemo -f
``` 

## Tag and push the image
```shell
docker tag aik8s-api:v5 <NameofACR>.azurecr.io/aik8s-api:v5 
docker push <NameofACR>.azurecr.io/aik8s-api:v5 
```

## Deploy to Kubernetes Cluster
Now that the image is in the container registry, it is time to deploy it to Kubernetes.
1. Deploy to a dedicated namespace
    You could use the default namespace, but it is recommended to put the test application in a dedicated one, for example, 'ai-k8s-demo'. To deploy a namespace, use content in [k8s-namespace.yaml](/k8s/k8s-namespace.yaml):
    ```shell
    kubectl apply -f k8s\k8s-namespace.yaml # tweak the path accordingly
    ```
2. Setup proper role binding for RBAC-enabled clusters
    If you have RBAC enabled for your cluster, permissions need to be granted to the service account to access the cluster info for telemetries. Refer to [Configure RBAC permissions](./k8s//configure-rbac-permissions.md) for details.
    For example, deploy a role assignment in the namespace of `aik8s-demo` by using [sa-role.yaml](/k8s/sa-role.yaml):
    ```shell
    kubectl apply -f k8s\sa-role.yaml # tweak the path accordingly
    ```
    _Notes: Check the namespace for the service account in the yaml file._
3. Deploy Secret with Application Insights information
    For example deploy secret by using [secretai.yaml](/k8s/secretai.yaml)
    ```shell
    kubectl create -f k8s\secretai.yaml # tweak the path accordingly
    ```
4. Deploy the application
    Create a Kubernetes deployment file. Reference [appdeploy.yaml](/k8s/appdeploy.yaml) as an example, pay attention to **namespace**, **image**, and **environment variables**, making sure they are properly set up.
    Then run the following command to deploy the app:
    ```shell
    kubectl apply -f k8s\appdeploy.yaml
    ```
5. Check out the pod deployed:
    ```shell
    kubectl get pods --namespace aik8s-demo    # Or whatever your namespace of choice.
    ```
    And you will see something like this:
    ```
    PS > kubectl get pods --namespace aik8s-demo
    NAME                                             READY   STATUS    RESTARTS   AGE
    some-other-pod-that-has-run-for-10-days-pmfdc    1/1     Running   0          10d
    aik8s-api-9c98784bb-9rzzn                        1/1     Running   0          52m
    ```
    Check out the logs, for example:
    ```shell
    PS > kubectl logs aik8s-api-9c98784bb-9rzzn --namespace aik8s-demo
    [Information] [2022-08-23T21:32:01.0135029Z] [CGroupContainerIdProvider] Got container id: 41a149977b4ea424947d54c59e7f63d32664e19503adda762004250f72db7687
    [Information] [2022-08-23T21:32:01.2737318Z] Found pod by name providers: ai-k8s-web-api-deployment-97d688b46-7cxs5
    info: Microsoft.Hosting.Lifetime[14]
        Now listening on: http://[::]:80
    info: Microsoft.Hosting.Lifetime[0]
        Application started. Press Ctrl+C to shut down.
    info: Microsoft.Hosting.Lifetime[0]
        Hosting environment: Production
    info: Microsoft.Hosting.Lifetime[0]
        Content root path: /app/
    ```
