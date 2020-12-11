# Demo of how to upgrade Knative Serving from 0.15.0 to 0.18.1 with Knative operator

## Prerequisites

### [Install Istio](https://istio.io/latest/docs/setup/getting-started/)

1. Download Istio 1.7.4

```
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.7.4 sh -
cd istio-1.7.4
```

2. Export the Istioctl

```
export PATH=$PWD/bin:$PATH
```

3. Install Istio Operator

```
istioctl operator init
```

Check all istio operator:

```
kubectl get deploy -n istio-operator
```

Check the log:
```
kubectl logs -f deploy/istio-operator -n istio-operator
```

4. Create the namespace and the Istio Operator CR

```
kubectl create ns istio-system
kubectl apply -f - <<EOF
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  namespace: istio-system
  name: example-istiocontrolplane
spec:
  profile: demo
EOF
```

Check the istio operator CR:

```
kubectl get IstioOperator example-istiocontrolplane -n istio-system
```

Check all istio deployments:
```
kubectl get deploy -n istio-system
```

Check all istio pod:
```
kubectl get pod -n istio-system
```


### Install Knative Serving 0.15.0 with Knative Operator 0.15.0

1. Install Knative Operator 0.15.0

```
kubectl apply -f https://github.com/knative/operator/releases/download/v0.15.0/operator.yaml
```

Check all the knative serving deployments:
```
kubectl get deploy
```

```
kubectl get pod -w
```

Check the log:
```
kubectl logs -f deploy/knative-operator
```

2. Create the Knative Serving CR:
```
kubectl create ns knative-serving
kubectl apply -f - <<EOF
apiVersion: operator.knative.dev/v1alpha1
kind: KnativeServing
metadata:
  name: knative-serving
  namespace: knative-serving
EOF
```

Check all the knative serving deployments:
```
kubectl get deploy -n knative-serving
```

Check the knative serving CR:
```
kubectl get KnativeServing knative-serving -n knative-serving
```


## Upgrade Knative Serving from 0.15.0 to 0.18.1

Install Knative Operator 0.18.1

```
kubectl apply -f https://github.com/knative/operator/releases/download/v0.18.1/operator.yaml
```

Check istio operator:
```
kubectl get deploy
```

Create a namespace `knative-serving`:
```
kubectl create namespace knative-serving
```

Install Knative Serving 0.16.0:
```
kubectl apply -f - <<EOF
apiVersion: operator.knative.dev/v1alpha1
kind: KnativeServing
metadata:
  name: knative-serving
  namespace: knative-serving
spec:
  version: 0.19
EOF
```

Install Knative Serving 0.17.0:
```
kubectl apply -f - <<EOF
apiVersion: operator.knative.dev/v1alpha1
kind: KnativeServing
metadata:
  name: knative-serving
  namespace: knative-serving
spec:
  version: 0.17.0
EOF
```

Install Knative Serving 0.18.1:
```
kubectl apply -f - <<EOF
apiVersion: operator.knative.dev/v1alpha1
kind: KnativeServing
metadata:
  name: knative-serving
  namespace: knative-serving
spec:
  version: 0.18.1
EOF
```


## Upgrade Knative Serving

Install Knative Operator 0.18.1:

```
kubectl apply -f https://github.com/knative/operator/releases/download/v0.18.1/operator.yaml
```

Upgrade from 0.15.0 to 0.16.0
```
kubectl apply -f - <<EOF
apiVersion: operator.knative.dev/v1alpha1
kind: KnativeServing
metadata:
  name: knative-serving
  namespace: knative-serving
spec:
  version: 0.16.0
EOF
```

Upgrade from 0.16.0 to 0.17.0
```
kubectl apply -f - <<EOF
apiVersion: operator.knative.dev/v1alpha1
kind: KnativeServing
metadata:
  name: knative-serving
  namespace: knative-serving
spec:
  version: 0.17.0
EOF
```

Upgrade from 0.17.0 to 0.18.1
```
kubectl apply -f - <<EOF
apiVersion: operator.knative.dev/v1alpha1
kind: KnativeServing
metadata:
  name: knative-serving
  namespace: knative-serving
spec:
  version: 0.18.1
EOF
```

Check the log:
```
kubectl logs -f deploy/knative-operator
```

Check all the knative serving pods:
```
kubectl get pod -n knative-serving
```

Check all the knative serving deployments:
```
kubectl get deploy -n knative-serving
```

Check the knative serving CR:
```
kubectl get KnativeServing knative-serving -n knative-serving
```


After it is downloaded, we will only focus on changing the part of the deployment.

### Create a ConfigMap for your own serving manifest

Open a terminal, and go to the directory of knative-serving-operator/install-custom-serving-manifest.
We can create a ConfigMap called config-manifest against your Kubernetes cluster with the following command:
```aidl
kubectl create configmap config-manifest --from-file=custom-serving-manifest.yaml
```

The name "custom-serving-manifest.yaml" will become a valid key available in the data of the above ConfigMap.

## Define the environment variable of KO_DATA_PATH

The operator will first search if there is any serving manifest files available under KO_DATA_PATH. If
this path is not available, the operator will pick up the one as packaged in the operator image.

The custom manifest must be put in a directory called knative-serving, so the deployment of the operator can be changed
into the following:
```aidl
apiVersion: apps/v1
kind: Deployment
metadata:
  name: knative-serving-operator
spec:
  replicas: 1
  selector:
    matchLabels:
      name: knative-serving-operator
  template:
    metadata:
      annotations:
        sidecar.istio.io/inject: "false"
      labels:
        name: knative-serving-operator
    spec:
      containers:
      - env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: OPERATOR_NAME
          value: knative-serving-operator
        - name: SYSTEM_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: METRICS_DOMAIN
          value: knative.dev/serving-operator
        # Add KO_DATA_PATH env variable
        - name: KO_DATA_PATH
          value: /etc/kodata
        image: gcr.io/knative-releases/knative.dev/serving-operator/cmd/manager@sha256:7008dfe31aee9bc7c81ea44171f8ab54f8289e2377e378837bba5f44af6f92c9
        imagePullPolicy: IfNotPresent
        name: knative-serving-operator
        # Add volumeMount
        volumeMounts:
          - name: config-manifest-volume
            mountPath: /etc/kodata/knative-serving
      # Add volume
      volumes:
        - name: config-manifest-volume
          configMap:
            name: config-manifest
      serviceAccountName: knative-serving-operator
```

As you see, we add an environment variable KO_DATA_PATH with the value /etc/kodata, create a volume called config-manifest-volume
linking to the ConfigMap config-manifest we just created, and add the volume mount at the path /etc/kodata/knative-serving.

This modified section of operator deployment is also available in the file [example-operator.yaml](example-operator.yaml).

### Install the operator

You need to run the Kubernetes command to apply the modified deployment section to install the operator:
```aidl
kubectl apply -f example-operator.yaml
```

If everything goes correct, you should be able to see a knative serving operator pod is successfully launched.

Attention: you need to be responsible for any change on the custom serving manifest. Please make sure it really works
for your designated Kubernetes cluster from your cloud provider.
