## Google-microservice with pyroscope

To use Google-microservice with pyroscop we need to do few steps mentioned below 
We need to add Pyroscope to the cluster.
we are using minikube cluster. so make sure you have install and configures Minikube.
To install pyroscope easily we are going to use helm chart.

# Pyroscope Helm Chart
[Helm](https://helm.sh) must be installed to use the chart.
Please refer to Helm's [documentation](https://helm.sh/docs/) to get started.

### Get the Repo of Pyrscope

```console
helm repo add pyroscope-io https://pyroscope-io.github.io/helm-chart
```

### Installing the Chart

To install the chart with the release name `pyroscope`:

```console

helm install pyroscope pyroscope-io/pyroscope 
```
OR

To access the Pyroscope using the NodePort

```console
helm install pyroscope pyroscope-io/pyroscope --set service.type=NodePort
```

### Docker Images
After installing the Pyroscope, Images need to be added,
images are created using the docker file which is stored inside the SRC folder. 
depending upon the application need to add pyroscope dependency

adding into Dockerfile
```console
# this copies pyroscope binary from pyroscope image to your image:
COPY --from=pyroscope/pyroscope:latest /usr/bin/pyroscope /usr/bin/pyroscope

# this starts your app with pyroscope profiler, make sure to change "python", "email_server.py" to the actual command
CMD [ "pyroscope", "exec", "python", "email_server.py ]
```

Now we have to build the image 
We will take email service as an example here:

```console
docker build . -t emailservice
```
Change the tag to push into your repository

Once that is done we will now move on to kubernetes-manifest folder
This folder includes all the yaml files for kubernetes,

emailservice.yaml 
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: emailservice
spec:
  selector:
    matchLabels:
      app: emailservice
  template:
    metadata:
      labels:
        app: emailservice
    spec:
      serviceAccountName: default
      terminationGracePeriodSeconds: 5
      containers:
      - name: server
        image: beellzrocks/emailservice  #Image which is build and uploaded to dockerhub. Change once you put it your repository.
        #imagePullPolicy: Never
        ports:
        - containerPort: 8080
        env:
        - name: PORT
          value: "8080"
        - name: PYROSCOPE_SERVER_ADDRESS  #to change the pyroscope server port chnage the value
          value: "http://pyroscope:4040"
        - name: PYROSCOPE_APPLICATION_NAME #to Change the application name shown in pyroscope
          value: "email.service"  
        # - name: DISABLE_TRACING
        #   value: "1"
        - name: DISABLE_PROFILER
          value: "1"
        readinessProbe:
          periodSeconds: 5
          exec:
            command: ["/bin/grpc_health_probe", "-addr=:8080"]
        livenessProbe:
          periodSeconds: 5
          exec:
            command: ["/bin/grpc_health_probe", "-addr=:8080"]
        securityContext: # this is added as pyroscope need it run properly 
          capabilities:
            add:
            - SYS_PTRACE  
        resources:
          requests:
            cpu: 100m
            memory: 64Mi
          limits:
            cpu: 200m
            memory: 128Mi
---
apiVersion: v1
kind: Service
metadata:
  name: emailservice
spec:
  type: ClusterIP
  selector:
    app: emailservice
  ports:
  - name: grpc
    port: 5000
    targetPort: 8080


```

These changes can be done here individually. 
Will have to change the other yaml files to fit your need.

### Applying the manifest 

After doing the changes you can use this command to create all the yaml files

```console
kubectl apply -f kubernetes-manifests/
```

The changes have to be made for the kubernetes-manifest.yaml file which is located in 
Release -> Kubernetes-manifest.yaml
It contains all the config in one file. you will need to change it according to the above mention process.

```console
kubectl apply -f release/kubernetes-manifests.yaml
```
To view all the pods:

```console
kubectl get pods
```
To view Pyroscope UI

```console 
minikube service pyroscope
```


#### Conclusion

We have installed and configured the pyroscope, also connected Google Microservice with pyroscope.
