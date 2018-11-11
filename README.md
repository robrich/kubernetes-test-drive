Kubernetes Test Drive
=====================

This code is the demos for the [Kubernetes Test Drive](https://robrich.org/slides/kubernetes-test-drive/#/) presentation.

The code
--------

In the presentation we dig into each file.  Wander through all the code in the `blue` folder to see behind the curtain.

The demos
---------

```bash
cd blue

docker build -t kubedemo:v1 src

docker run -p 5000:80 kubedemo:v1
docker container stop ...
# open browser to localhost:5000
docker container list
docker container stop ... # <-- include container name
docker container rm ... # <-- include container name
```

We ran the docker container.  This didn't use Kubernetes at all.

```bash
kubectl cluster-info
kubectl apply -f kube/pod.yaml
kubectl get pods
kubectl describe pods kubedemo
```

We have a pod running, but we can't browse to it.

```bash
kubectl port-forward kubedemo 5000:80
```

We can now browse to `localhost:5000` to see the container.
This is the "cheater" way.

Stop the terminal, and let's kill this pod.

```bash
kubectl delete -f kube/pod.yaml
kubectl get all
```

The pod is gone. Let's schedule a deployment and a service that points to it.

```bash
kubectl apply -f kube/deployment.yaml
kubectl apply -f kube/service.yaml
kubectl get all
```

We now have a deployment and a service pointing to the deployment.

```bash
# since it's type:NodePort
kubectl get all
# svc/kubedemo-service  NodePort  0.0.0  <none>  80:**31580**/TCP  1s
# grab 30,000 port
http://localhost:31580
```

The product is now running.  We can browse to it and see the website.

Now let's imagine we have a requirements change.  Normally this would be done in the same folder, but let's switch to the `green` folder and "change" content there.

```bash
cd ..
cd green
```

We modified index.html.  Let's build the next version of the docker image.

```bash
docker build -t kubedemo:v2 src
# We've changed kube/deployment.yaml to reference v2 in all 3 places.
kubectl apply -f kube/deployment.yaml
kubectl get all # <-- run this quickly a bunch of times and you might catch the scale up
# refresh the page
```

The website didn't go down, and we now have the new version running.  Score!  Zero down-time deployment.

Other useful commands:

```bash
# get deployment details
kubectl get all
kubectl describe po/kubedemo-deployment-... # <-- pick a pod from the list above
# get logs for each
kubectl get all
kubectl logs po/kubedemo-deployment-...
```

Time to clean up:

```bash
cd ..

kubectl delete -f blue/kube/service.yaml
kubectl delete -f green/kube/deployment.yaml

kubectl get all
```

The only service left is the Kubernetes API.
