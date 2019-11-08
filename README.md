# Demos of Knative Operators

Knative community founded an [operation work group](https://knative.dev/contributing/working-groups/#operations) to resolve the operational issues during Knative operation
work. There are so far two operators already created: [Knative Serving Operator](https://github.com/knative/serving-operator/) and [Knative Eventing Operator](https://github.com/knative/eventing-operator/).
One of the goals for [Knative operation work group](https://knative.dev/contributing/working-groups/#operations) is to make the operator the default way to install Knative
components, including Serving and Eventing.

Since it is still in the early stage of developing Knative operators, and we have successfully released Knative
Serving Operator and Knative Eventing Operator. This repository is going to provide useful demos about how
you can use the Knative operators and what you can do with the them.

There are two directories in this repository: [knative-serving-operator](knative-serving-operator) and [knative-eventing-operator](knative-eventing-operator), where
you can find demos respectively for each operator.

## How Operator Works

If you would like to know more about Kubernetes operator and Knative operators, you are welcome to read
articles listed as below:

- [The definition and importance of Kubernetes Operator](https://medium.com/@vincenthou/kubernetes-operator-journey-ep1-the-definition-and-importance-of-kubernetes-operator-23fc88010dbf)
- [Process to build a Kubernetes Operator](https://medium.com/@vincenthou/kubernetes-operator-journey-ep2-process-to-build-a-kubernetes-operator-9747f060e2e8)
- [How to create a Kubernetes Operator based on Knative common packages](https://medium.com/@vincenthou/kubernetes-operator-journey-ep3-how-to-create-a-kubernetes-operator-based-on-knative-common-8d344caf9eea)
- [Controller-runtime VS Knative/pkg for operator](https://medium.com/@vincenthou/kubernetes-operator-journey-ep4-controller-runtime-vs-knative-pkg-for-operator-b71468b23a37)
- [What upgrade means and how to upgrade the operator](https://medium.com/@vincenthou/kubernetes-operator-journey-ep5-what-upgrade-means-and-how-to-upgrade-the-operator-daa21cafee31)

You will grasp how Knative operators are built, after you go through these materials.

As always, follow Vincent, you won't derail!
