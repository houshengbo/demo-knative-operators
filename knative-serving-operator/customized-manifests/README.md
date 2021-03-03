# Demo of leveraging customized manifests to configure the security context for the deployment

The file activator-0.21.yaml contains the expected resource with the expected security context.
It has been published at `https://github.com/houshengbo/demo-knative-operators/releases/download/master/activator-0.21.yaml`

In addition, I would like to install network certificate manager as well.

Apply the serving CR to install serving with the expected resource.

```aidl
kubectl apply -f knative-serving-customized-additional.yaml
```
