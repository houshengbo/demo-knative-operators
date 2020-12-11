# Demo of leveraging customized manifests to install other ingresses

## Use Kourier instead of istio

Check the content of `knative-serving-customized-kourier.yaml`.

The network ingress plugin is installed under the same namespace as the Knative Serving.

Fetch the External IP or CNAME:

```
kubectl --namespace knative-serving get service kourier
```


```
kn service create hello \
      --image gcr.io/knative-samples/helloworld-go \
      --env TARGET=Knative
```