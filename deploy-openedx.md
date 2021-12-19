
# Deploy openedx through tutor 

In the terminal run 
```sh 
tutor k8s quickstart
```
*PS. the quickstart for tutor could fail for the time because it will wait until the dbs pods get ready which could take a while*

In this case, you can run 
```sh 
tutor k8s start 
``` 
After that, check when all the pods are ready via 
```sh 
kubectl get pods -n openedx
```
or through the dashboard 

when everything is fine, you can run 
```sh 
tutor k8s init 
```
Apply the nginx ingress controller:

```sh 
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/mandatory.yaml 

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/provider/cloud-generic.yaml 
```

apply the nginx-ingress:

```sh 
kubectl -n openedx  apply -f ingress.yml 
```
**PS: the host used in the ingress-file are:**

 * ecme.io: for the lms
 * cms.io: for the cms 

**make sure that the tutor config file get the same value**
