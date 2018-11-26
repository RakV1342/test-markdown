Monitoring NetScaler Appliances in Kubernetes
===

This document describes how the [NetScaler Metrics Exporter](https://github.com/citrix/netscaler-metrics-exporter) and [Prometheus-Operator](https://github.com/coreos/prometheus-operator) can be used to monitor VPX/CPX ingress devices and CPX-EW (east-west) devices.


Launching Promethus-Operator
---
Prometheus Operator has an expansive method of monitoring services on Kubernetes. To get started quickly, this guide uses [kube-prometheus](https://github.com/coreos/prometheus-operator/tree/master/contrib/kube-prometheus) and its [manifest files](https://github.com/coreos/prometheus-operator/tree/master/contrib/kube-prometheus/manifests).
The manifest files help deploy a basic working model of Prometheus Operator with a simple quick command;
```
kubectl create -f prometheus-operator/contrib/kube-prometheus/manifests/
```
This creates several pods and services, of which ```prometheus-k8s-xx``` pods are for metrics aggregation and timestamping and ```grafana``` pods for visualization. An output similar to this should be seen;
```
$ kubectl get pods -n monitoring
NAME                                   READY     STATUS    RESTARTS   AGE
alertmanager-main-0                    2/2       Running   0          2h
alertmanager-main-1                    2/2       Running   0          2h
alertmanager-main-2                    2/2       Running   0          2h
grafana-5b68464b84-5fvxq               1/1       Running   0          2h
kube-state-metrics-6588b6b755-d6ftg    4/4       Running   0          2h
node-exporter-4hbcp                    2/2       Running   0          2h
node-exporter-kn9dg                    2/2       Running   0          2h
node-exporter-tpxhp                    2/2       Running   0          2h
prometheus-k8s-0                       3/3       Running   1          2h
prometheus-k8s-1                       3/3       Running   1          2h
prometheus-operator-7d9fd546c4-m8t7v   1/1       Running   0          2h
```
**NOTE:** It may be preferable to expose the Prometheus and Grafana pods via NodePorts. To do so, the prometheus-service.yaml and grafana-service.yaml files will need to be modified as follows;


<details>
<summary>prometheus-service.yaml</summary>
<br>

```
  ADD
  ADD
  ADD
```
To apply these changes into the kubernetes cluseter run: ```kubectl apply -f prometheus-service.yaml```.

</details>


<details>
<summary>grafana-service.yaml</summary>
<br>

```
  ADD
  ADD
  ADD
```
To apply these changes into the kubernetes cluseter run: ```kubectl apply -f grafana-service.yaml```.

</details>



Configuring Netscaler Metrics Exporter
---
This section describes how to integrate the Netscaler Metrics Exporter with the VPX/CPX ingress or CPX-EW devices. 

<details>
<summary>VPX Ingress Device</summary>
<br>

To monitor an ingress VPX device, the netscaler-metrics-exporter will be run as a pod within the kubernetes cluster. The IP of the VPX ingress device will be provided as an argument to the exporter. An example yaml file to deploy such an exporter is given below;

```
apiVersion: v1
kind: Pod
metadata:
  name: exp
  labels:
    app: exp
spec:
  containers:
    - name: exp
      image: ns-exporter:v1
      args:
        - "--target-nsip=<IP_and_port_of_VPX>"
        - "--port=8888"
      imagePullPolicy: IfNotPresent
---
apiVersion: v1
kind: Service
metadata:
  name: exp
  labels:
    app: exp
spec:
  type: ClusterIP
  ports:
  - port: 8888
    targetPort: 8888
    name: exp-port
  selector:
    app: exp
```
The IP and port of the VPX device needs to be filled in as the ```--target-nsip``` (Eg. ```--target-nsip=10.0.0.20```). 
</details>

<details>
<summary>CPX Ingress Device</summary>
<br>
  
To monitor a CPX ingress device, the exporter is added as a side-car. An example yaml file of a CPX ingress device with an exporter as a side car is given below;
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: cpx-ingress
  labels:
    name: cpx-ingress
spec:
  replicas: 1
  selector:
    matchLabels:
      name: cpx-ingress
  template:
    metadata:
      labels:
        name: cpx-ingress
      annotations:
        NETSCALER_AS_APP: "True"
    spec:
      serviceAccountName: cpx
      containers:
        - name: cpx-ingress
          image: "us.gcr.io/citrix-217108/citrix-k8s-cpx-ingress:latest"
          securityContext:
            privileged: true
          env:
            - name: "EULA"
              value: "YES"
            - name: "NS_PROTOCOL"
              value: "HTTP"
            #Define the NITRO port here
            - name: "NS_PORT"
              value: "9080"
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
            - name: nitro-http
              containerPort: 9080
            - name: nitro-https
              containerPort: 9443
        # Add exporter as a sidecar
        - name: exporter
          image: ns-exporter:v1
          args:
            - "--target-nsip=192.0.0.2:80"
            - "--port=8888"
          imagePullPolicy: IfNotPresent      
---
kind: Service
apiVersion: v1
metadata:
  name: cpx-ingress
  labels:
    name: cpx-ingress
spec:
  selector:
    name: cpx-ingress
  ports:
    - name: http
      port: 80
      targetPort: http
    - name: https
      port: 443
      targetPort: https
    # Expose exporter as a k8s service
    - name: exp-port
      port: 8888
      targetPort: 8888
```
Here, the exporter uses the ```192.0.0.2``` local IP to fetch metrics from the CPX.

</details>


<details>
<summary>CPX-EW Dvice</summary>
<br>
To monitor a CPX-EW (east-west) device, the exporter is added as a side-car. An example yaml file of a CPX-EW device with an exporter as a side car is given below;
```
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: cpx
spec:
  template:
    metadata:
      name: cpx
      labels:
        app: cpx-daemon
      annotations:
        NETSCALER_AS_APP: "True"
    spec:
      serviceAccountName: cpx
      hostNetwork: true
      containers:
        - name: cpx
          image: "in-docker-reg.eng.citrite.net/cpx-dev/cpx:12.1-48.118"
          securityContext: 
             privileged: true
          env:
          - name: "EULA"
            value: "yes"
          - name: "NS_NETMODE"
            value: "HOST"
          #- name: "kubernetes_url"
          #  value: "https://10.106.76.232:6443"
        # Add exporter as a sidecar
        - name: exporter
          image: ns-exporter:v1
          args:
            - "--target-nsip=192.168.0.2:80"
            - "--port=8888"
          imagePullPolicy: IfNotPresent      
---
kind: Service
apiVersion: v1
metadata:
  name: cpx-ew
  labels:
    name: cpx-ew
spec:
  selector:
    name: cpx-ew
  ports:
    # Expose exporter as a k8s service
    - name: exp-port
      port: 8888
      targetPort: 8888
```
Here, the exporter uses the ```192.168.0.2``` local IP to fetch metrics from the CPX.

</details>



Service Monitors to Detect Netscaler Devices
---
The netscaler metrics exporters helps collect data from the VPX/CPX ingress and CPX-EW devices. This exporters needs to be detected by Prometheus Operator so that the metrics can be timestamped, stored, and exposed for visualization on Grafana. Prometheus Operator uses the concept of ```ServiceMonitors``` to detect pods belonging to a service, using the labels attached to that service. 

The following example yaml file will detect all the exporter services (given in the example yaml files above) which have the label ```service: citrix-adc``` associated with them. 

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    k8s-app: rak-app
  name: rak-app
  namespace: monitoring
spec:
  endpoints:
  - interval: 30s
    port: exp-port
  jobLabel: k8s-app
  selector:
    matchLabels:
      k8s-app: rak-app
```


Verification
---
Prometheus Operator will take a few minutes to detect the presence of the netscaler-metrics-exporter. After 3-5 minutes, it should appear in the ```Targets``` page;
<ADD>

Now metrics being collected from CPX by netscaler-metrics-exporter will be visible on Grafana as well.
<ADD>




