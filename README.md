Monitoring Citrix Ingress Controller and CPX-EW using NetScaler Metrics Exporter and Prometheus Operator
===

This document describes how the [NetScaler Metrics Exporter](https://github.com/citrix/netscaler-metrics-exporter) and [Prometheus-Operator](https://github.com/coreos/prometheus-operator) can be used to monitor ingress devices and CPX-EW devices.


Launching Promethus-Operator
---
Prometheus Operator has an expansive method of monitoring services on Kubernetes. To get started quickly, this guide uses [kube-prometheus](https://github.com/coreos/prometheus-operator/tree/master/contrib/kube-prometheus) and its [manifest files](https://github.com/coreos/prometheus-operator/tree/master/contrib/kube-prometheus/manifests).
The manifest files help deploy a basic working model of Prometheus Operator with a simple quick command;
```
kubectl create -f prometheus-operator/contrib/kube-prometheus/manifests/
```
This creates several pods and services, of which ```prometheus-k8s-xx``` pods are for metrics aggregation and timestamping and ```grafana``` pods. An output similar to this should be seen;
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

Expose prom-k8s and grafana as node port. 
ADD: prom-k8s targets page

Configuring Netscaler Metrics Exporter for Ingress Device
---
This section describes how to integrate the Netscaler Metrics Exporter with the VPX or CPX ingress device. 

<details>
<summary>VPX Ingress Device</summary>
<br>

To monitor a VPX device, the netscaler metrics exporter will be run a pod within the kubernetes cluster and the arguments fed to it will point to the IP of the VPX ingress device. The yaml file to deply such an exporter is given below:

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


</details>

<details>
<summary>CPX Ingress Device</summary>
<br>
add
  add
  add
</details>


Adding NetScaler Metrics Exporter as a Side-Car to CPX
---
With the Prometheus Operator (kube-promehteus) setup, the netscaler-metrics-exporter can be added as a sidecar to the CPX and then exposed to Prometheus Operator using ServiceMonitors.

Adding netscaler-metrics-exporter as a sidecar to CPX is simply the addition of a few lines to the ```containers:``` and ```Services``` sections of the CPX yaml file.


1. For CPX-Ingress
<ADD>
2. For CPX-EW
<ADD>


The service monitor to monitor the netscaler-metrics-exporter and intimate Prometheus Operator of its existance can be added using the yaml file given below;
```
<ADD>

```


Verification
---
Prometheus Operator will take a few minutes to detect the presence of the netscaler-metrics-exporter. After 3-5 minutes, it should appear in the ```Targets``` page;
<ADD>

Now metrics being collected from CPX by netscaler-metrics-exporter will be visible on Grafana as well.
<ADD>




