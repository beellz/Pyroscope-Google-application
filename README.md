

## Google-microservice with pyroscope
To use Google-microservice with pyroscope we need to do few steps mentioned below 

### Retagging Docker Images
Docker images need to build and pushed to repositories. (Currently, it is in DockerHub under beellzrocks)\
Dockerfile of each microservice is stored inside their respective folder under the src folder. \
To change the docker file and retagging it \
Follow these steps:

For example Email service
Go to src/emailservice

- add below lines in Dockerfile

```console
COPY --from=pyroscope/pyroscope:latest /usr/bin/pyroscope /usr/bin/pyroscope
CMD [ "pyroscope", "exec", "python", "email_server.py ]
```

- build Docker image inside the folder using Dockerfile
```console
docker build . -t <yourRepository/serviceName:version>
```
- Change the tag to push into your repository

```console
docker push <yourRepository/serviceName:version>
```


### Kubernetes manifest 
For the Email service we can find the yaml file inside:
kubernetes-manifests/emailservice.yaml

emailservice.yaml
```
   containers:
      - name: server
        image: beellzrocks/emailservice  #Change the image with your repository tag
```        

## Steps to install microservice and profile it using pyroscope

To install the pyroscope we are going to use the helm chart.\
 [Helm](https://helm.sh) must be installed to use the chart.\
Please refer to Helm's [documentation](https://helm.sh/docs/) to get started.

 Get the Repo of Pyrscope
```console
helm repo add pyroscope-io https://pyroscope-io.github.io/helm-chart
```
Installing the Chart
To install the chart with the release name `pyroscope`:
```console
helm install pyroscope pyroscope-io/pyroscope 
```

After doing the changes you can use this command to create all the yaml files
```console
kubectl apply -f kubernetes-manifests/
```
To view all the pods:
```console
kubectl get pods
```
To view Pyroscope UI
```console 
kubectl port-forward svc/pyroscope 8080:4040
```
Now you can access pyroscope UI on http://localhost:8080

## Changes we did to integrate microservices with pyroscope

For Python

Need to change in docker file:
```console
COPY --from=pyroscope/pyroscope:latest /usr/bin/pyroscope /usr/bin/pyroscope
CMD [ "pyroscope", "exec", "python", "email_server.py ]
```
build Docker image using Dockerfile
```console
docker build . -t <yourRepository/serviceName:version>
```
- Change the tag to push into your repository

```console
docker push <yourRepository/serviceName:version>
```

In the Kubernetes file under the kubernetes-manifests folder
emailservice.yaml

      containers:
      - name: server
        image: beellzrocks/emailservice  #Change the image with your repository image


For .Net

Need to change in docker file:
```
COPY --from=pyroscope/pyroscope:latest /usr/bin/pyroscope /usr/bin/pyroscope
ENTRYPOINT ["pyroscope", "exec", "-spy-name", "dotnetspy", "/app/cartservice"]
```
build Docker image using Dockerfile
```console
docker build . -t <yourRepository/serviceName:version>
```
- Change the tag to push into your repository

```console
docker push <yourRepository/serviceName:version>
```

For Go

Changes to be done in main.go file

```
import (
	pyroscope "github.com/pyroscope-io/pyroscope/pkg/agent/profiler"
)

func main() {

	pyroscope.Start(pyroscope.Config{
		ApplicationName: os.Getenv("APPLICATION_NAME"),
		ServerAddress:   os.Getenv("SERVER_ADDRESS"),
	})
	/ code here
)	
```

- Build Docker image using Dockerfile
```console
docker build . -t <yourRepository/serviceName:version>
```
- Change the tag to push into your repository

```console
docker push <yourRepository/serviceName:version>
```

## Microservices we will be profiling using pyroscope

Python:
* Recommendationservice	
* Emailservice

Go:
* Frontend
* Productcatalogservice
* Shippingservice
* Checkoutservice

.Net: 
* Cartservice