# Demo of basic features for Knative Serving Operator

You can find the official released artifacts of Knative Serving Operator at the [release page](https://github.com/knative/serving-operator/releases).
The latest release is v0.10.0.

## Install Serving Operator

Although Knative Serving supports multiple networking dependencies, Serving Operator currently still ask you
to install Istio as a prerequisite.

The following steps apply to Mac environment.

Install Helm:

```aidl
brew install kubernetes-helm
```

To upgrade Helm:

```aidl
brew upgrade kubernetes-helm
```

Download Istio and install CRDs

```aidl
export ISTIO_VERSION=1.1.7
curl -L https://git.io/getLatestIstio | sh -
cd istio-${ISTIO_VERSION}
for i in install/kubernetes/helm/istio-init/files/crd*yaml; do kubectl apply -f $i; done
```

Create istio-system namespace

There is a YAML file you can use to install the namespace for istio. If you have download this repository,
run the following commands:

```
cd demo-knative-operators
kubectl apply -f knative-serving-operator/dependencies/istio-namespace.yaml
```

Or directly apply the following command:

```aidl
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: istio-system
  labels:
    istio-injection: disabled
EOF
```




## Install Knative Serving Operator