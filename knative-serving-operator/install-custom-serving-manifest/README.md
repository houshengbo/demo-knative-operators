# Demo of how to install custom serving manifest

By default, the Knative serving mamifest, also known as the release file, has been packaged in the released
image of Knative operator. When you install the serving operator, it will parse and load the serving manifest,
in order to install the Knative serving.

However, the released Knative serving manifest is unable to fit in any cloud provider. You probably need
to tailor and install the serving manifest specially for your own cloud environment. There are two approaches
to install your own serving manifest:

* Regenerate the operator image with your own serving manifest from the source code, by replacing the YAML file
under [cmd/manager/kodata/knativeserving](https://github.com/knative/serving-operator/tree/master/cmd/manager/kodata/knative-serving) in the [serving-operator](https://github.com/knative/serving-operator) repository. 
* Load your own manifest by specifying the path to read the serving manifest.

This example will focus on the second way.

## Create the custom serving manifest for your cloud provider

You can name it after any name you want. Here we call it custom-serving-manifest.yaml. You need to prepare it for your cloud
with the customization. In this repository, I keep its content the same as in 0.10.0.yaml of serving.

## Download the release file of serving operator

The latest release of serving operator is 0.10.0. Run the following command to download it to your local machine. 

```aidl
wget -O example-operator.yaml https://github.com/knative/serving-operator/releases/download/v0.10.0/serving-operator.yaml
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
