## Minimum requirements to run spark on k8 cluster
### Step1: Start the minikube cluster

**Start minikube with at-least 8GB Ram and 4 Cores to run spark**
> minikube start --driver vmware --memory 8192 --cpus 4

**Start the dashboard**
> minikube dashboard

### Create ServiceAccount and Cluster-Role-Binding on K8
**Create Service Account on kubernetes for spark**
> kubectl create serviceaccount spark

**Create cluster-role-binding in default namespace**
> kubectl create clusterrolebinding spark-role --clusterrole=edit --serviceaccount=default:spark --namespace=default

### Create docker image of your application and push to Docker hub
1. Write your own Dockerfile using any base image. I used apache/spark as base image and copied my jar to the same.
</br>**Sample DockerFile Content**
```
FROM apache/spark
COPY <path of jar on local system>/<jar_file_name>.jar /opt/spark/work-dir
```
2. Build the image:
> docker build -t rajesh920352/spark-on-k8s:latest .

3. Push the image to your public dockerhub repository:
> docker push rajesh920352/spark-on-k8s:latest

### Get info to run spark-submit:
1. Get Kubernetes master hostname and port number detail using:
   > kubectl cluster-info
2. Your image name from DockerHub with the tag
3. Path of jar inside the docker image
4. Spark Kubernetes basic configurations


### Run spark-submit:

./bin/spark-submit \
--master k8s://<Kubernetes master hostname and port number>\
--deploy-mode cluster \
--name spark-pi \
--class org.apache.spark.examples.SparkPi \
--conf spark.executor.instances=2 \
--conf spark.kubernetes.container.image=apache/spark \
--conf spark.kubernetes.container.image.pullPolicy=Never \
--conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
local:///opt/spark/examples/jars/spark-examples_2.12-3.3.1.jar 10000000

### To see spark UI, enable port forwarding
kubectl port-forward <spark_driver_pod_name> 4040:4040

### To check the logs of spark driver
kubectl -n=default logs -f <spark_driver_pod_name>