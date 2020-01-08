# Demo of setting up custom ingress gateway

This demo will show you how to configure your custom ingress gateway with the Knative Serving operator.

## Create the service and the deployment

You probably need to define the Service and the Deployment of your ingress gateway. Here is the example:

```aidl
apiVersion: v1
kind: Service
metadata:
  name: custom-ingressgateway
  namespace: istio-system
  annotations:
  labels:
    chart: gateways-1.0.1
    release: RELEASE-NAME
    heritage: Tiller
    app: custom-ingressgateway
    custom: ingressgateway
spec:
  type: LoadBalancer
  selector:
    app: custom-ingressgateway
    custom: ingressgateway
  ports:
    - name: http2
      nodePort: 32380
      port: 80
      targetPort: 80
    - name: https
      nodePort: 32390
      port: 443
    - name: tcp
      nodePort: 32400
      port: 31400
    - name: tcp-pilot-grpc-tls
      port: 15011
      targetPort: 15011
    - name: tcp-citadel-grpc-tls
      port: 8060
      targetPort: 8060
    - name: tcp-dns-tls
      port: 853
      targetPort: 853
    - name: http2-prometheus
      port: 15030
      targetPort: 15030
    - name: http2-grafana
      port: 15031
      targetPort: 15031
---
# This is the corresponding deployment to back the gateway service
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: custom-ingressgateway
  namespace: istio-system
  labels:
    chart: gateways-1.0.1
    release: RELEASE-NAME
    heritage: Tiller
    app: custom-ingressgateway
    custom: ingressgateway
spec:
  replicas: 1
  selector:
    matchLabels:
      app: custom-ingressgateway
      custom: ingressgateway
  template:
    metadata:
      labels:
        app: custom-ingressgateway
        custom: ingressgateway
      annotations:
        sidecar.istio.io/inject: "false"
        scheduler.alpha.kubernetes.io/critical-pod: ""
    spec:
      serviceAccountName: istio-ingressgateway-service-account
      containers:
        - name: istio-proxy
          image: "docker.io/istio/proxyv2:1.0.2"
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
            - containerPort: 443
            - containerPort: 31400
            - containerPort: 15011
            - containerPort: 8060
            - containerPort: 853
            - containerPort: 15030
            - containerPort: 15031
          args:
            - proxy
            - router
            - -v
            - "2"
            - --discoveryRefreshDelay
            - "1s" #discoveryRefreshDelay
            - --drainDuration
            - "45s" #drainDuration
            - --parentShutdownDuration
            - "1m0s" #parentShutdownDuration
            - --connectTimeout
            - "10s" #connectTimeout
            - --serviceCluster
            - custom-ingressgateway
            - --zipkinAddress
            - zipkin:9411
            - --statsdUdpAddress
            - istio-statsd-prom-bridge:9125
            - --proxyAdminPort
            - "15000"
            - --controlPlaneAuthPolicy
            - NONE
            - --discoveryAddress
            - istio-pilot:8080
          resources:
            requests:
              cpu: 10m
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.namespace
            - name: INSTANCE_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.podIP
            - name: ISTIO_META_POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
          volumeMounts:
            - name: istio-certs
              mountPath: /etc/certs
              readOnly: true
            - name: ingressgateway-certs
              mountPath: "/etc/istio/ingressgateway-certs"
              readOnly: true
            - name: ingressgateway-ca-certs
              mountPath: "/etc/istio/ingressgateway-ca-certs"
              readOnly: true
      volumes:
        - name: istio-certs
          secret:
            secretName: istio.istio-ingressgateway-service-account
            optional: true
        - name: ingressgateway-certs
          secret:
            secretName: "istio-ingressgateway-certs"
            optional: true
        - name: ingressgateway-ca-certs
          secret:
            secretName: "istio-ingressgateway-ca-certs"
            optional: true
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: beta.kubernetes.io/arch
                    operator: In
                    values:
                      - amd64
                      - ppc64le
                      - s390x
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 2
              preference:
                matchExpressions:
                  - key: beta.kubernetes.io/arch
                    operator: In
                    values:
                      - amd64
            - weight: 2
              preference:
                matchExpressions:
                  - key: beta.kubernetes.io/arch
                    operator: In
                    values:
                      - ppc64le
            - weight: 2
              preference:
                matchExpressions:
                  - key: beta.kubernetes.io/arch
                    operator: In
                    values:
                      - s390x
```

Again, based on different cloud providers you use, the service and deployment may vary, but make sure
you create the correct ones for your cluster.

## Update the ConfigMap and the Gateway with the custom resource

As you see, we use `custom: ingressgateway` as one of the match labels, and the deployment is named `custom-ingressgateway`.
We are going to change the operator CR based on these two names. 

There are some changes you need to do with the CR to be created or updated. We need to change the label selector
of the gateway to `custom: ingressgateway`, and update the istio ConfigMap by setting the value
`custom-ingressgateway.istio-system.svc.cluster.local` for the key `gateway.knative-ingress-gateway` as below:

```aidl
apiVersion: v1
kind: Namespace
metadata:
  name: knative-serving
---
apiVersion: operator.knative.dev/v1alpha1
kind: KnativeServing
metadata:
  name: knative-serving
  namespace: knative-serving
spec:
  knative-ingress-gateway:
    selector:
      custom: ingressgateway
  config:
    istio:
      gateway.knative-serving.knative-ingress-gateway: "custom-ingressgateway.istio-system.svc.cluster.local"
```

You can access this file [here](example-cr-custom-ingress-gateway.yaml). By applying this yaml file, you
are able to set up your custom ingress gateway.
