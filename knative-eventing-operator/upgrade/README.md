# Demo of upgrade for Knative Eventing Operator

You can find the official released artifacts of Knative Eventing Operator at the [release page](https://github.com/knative/eventing-operator/releases).
The latest release is v0.10.0.

## Create a test namespace for eventing operator

We use the namespace test-eventing.

```aidl
kubectl create namespace test-eventing
```

## Install Eventing Operator v0.10.0

Set an environment variable as the test namespace:

```aidl
export TEST_NAMESPACE=test-eventing
```

The current release of Eventing Operator is v0.10.0. You can run:

```aidl
kubectl -n $TEST_NAMESPACE apply -f https://github.com/knative/eventing-operator/releases/download/v0.10.0/eventing-operator.yaml
```

### Verify the installation of Eventing Operator

Run the following command to see the operator pod:

```aidl
kubectl -n $TEST_NAMESPACE get pod
```

## Install Eventing by creating operator custom resource

The CR of eventing operator controls everything. When CR is created, the eventing will be installed. The
YAML file [knative-eventing-cr-0.10.yaml](knative-eventing-cr-0.10.yaml) is for you to use.

Go to the home directory of this repository, and run:
```aidl
kubectl -n $TEST_NAMESPACE apply -f knative-eventing-operator/upgrade/knative-eventing-cr-0.10.yaml
```

You should have Knative Eventing v0.10.0 installed. Check the CR of knative eventing:
```aidl
kubectl get Eventing knative-eventing -n $TEST_NAMESPACE -oyaml
```

You will see the version in the status is set to 0.10.0.

## Upgrade to Knative Eventing Operator 0.11.0 (with the PR [53](https://github.com/knative/eventing-operator/pull/53))

Go to the directory of the root directory of eventing-operator, make sure this PR is downloaded to your local machine.
Install the Knative Eventing operator from the source code by the command:

```aidl
ko -n $TEST_NAMESPACE apply -f config/
```

You will see a new pod named "ke-version-controller" has been launched.

Upgrade your operator and Knative Eventing from 0.10.0 to 0.11.0 with the CR of KEVersionController.
You can find the example [here](knative-eventing-upgrade-cr-to-0.11.yaml).

Before upgrading the operator with the creation of the version controller CR, we suggest you trace
the log of the version controller pod and the knative eventing operator pod, so that you can see what
will change.

Upgrade the version to 0.11.0 by creating this version controller CR by the command:
```aidl
kubectl -n $TEST_NAMESPACE apply -f knative-eventing-operator/upgrade/knative-eventing-upgrade-cr-to-0.11.yaml
```

You should be able to see the knative eventing operator has been upgraded. Check the CR of knative eventing:
```aidl
kubectl get KnativeEventing knative-eventing -n $TEST_NAMESPACE -oyaml
```

You will see the version in the status is changed to 0.11.0.
