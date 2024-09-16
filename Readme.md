# Istio Setup

### Download Istio contents using below commands and set the path in bin
```console
curl -L https://istio.io/downloadIstio | sh -
export PATH=$PWD/bin:$PATH
```
### Install Istio using istioctl
```console
istioctl install --set profile=demo -y
```
### We will need to change the ingressgateway service type from LoadBalancer to NodePort
```console
kubectl patch svc istio-ingressgateway -n istio-system -p '{"spec": {"type": "NodePort"}}'
```
### For this exercise we will consider example of bookinfo app
#### The application displays information about a book, similar to a single catalog entry of an online book store. Displayed on the page is a description of the book, book details (ISBN, number of pages, and so on), and a few book reviews.

#### The Bookinfo application is broken into four separate microservices:

#### productpage. The productpage microservice calls the details and reviews microservices to populate the page.
#### details: The details microservice contains book information.
#### reviews: The reviews microservice contains book reviews. It also calls the ratings microservice.
#### ratings: The ratings microservice contains book ranking information that accompanies a book review.
#### There are 3 versions of the reviews microservice:
![bookinfo app architecture](withistio.svg)
#### Version v1 doesn’t call the ratings service.
#### Version v2 calls the ratings service, and displays each rating as 1 to 5 black stars.
#### Version v3 calls the ratings service, and displays each rating as 1 to 5 red stars.

### Create a new namespace 
```console
kubectl create ns bookinfo
```
### Enable the namespace for Istio injection
```console
kubectl label namespace bookinfo istio-injection=enabled
```
### Analyze the namespace
```console
istioctl analyze -n bookinfo
```
### Deploy the bookinfo app
```console
kubectl apply -f bookinfo/bookinfo.yaml -n bookinfo
```
### To confirm that the Bookinfo application is running, send a request to it by a curl command from some pod, for example from ratings:
```console
kubectl exec <ratings-pod-name> -n bookinfo -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
```
### Now that the Bookinfo services are up and running, you need to make the application accessible from outside of your Kubernetes cluster, e.g., from a browser. A gateway is used for this purpose.
### We are also creating a virtual service.
```console
kubectl apply -f bookinfo/bookinfo-gateway.yaml -n bookinfo
```
### Confirm the gateway has been created
```console
kubectl get gateway
```
### To confirm that the Bookinfo application is accessible from outside the cluster, run the following curl command:
```console
export GATEWAY_URL=<node IP or host name>:<node-port>
curl -s "http://${GATEWAY_URL}/productpage" | grep -o "<title>.*</title>"
```
### Deploy Kiali & Prometheus using command below
```console
kubectl apply -f addons/kiali.yaml -n istio-system
kubectl apply -f addons/prometheus.yaml -n istio-system
```

### Try to open the UI of Kiali on Node port 30007
### Now lets generate some traffic on bookinfo application for testing
```console
while sleep 0.01; do curl -sS "http://${GATEWAY_URL}/productpage" &> /dev/null ; done
```
![Kiali UI of app bookinfo](kiali.png)