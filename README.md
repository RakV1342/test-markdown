# test-markdown


Exporter for NetScaler Stats
===

Description:
---

This is a simple server that scrapes Citrix NetScaler (NS) stats and exports them via HTTP to Prometheus. Prometheus can then be added as a data source to Grafana to view the netscaler stats graphically.

![exporter_diagram](https://user-images.githubusercontent.com/40210995/41391720-f89ee57e-6fb9-11e8-9550-02dc60dcfa43.png)

   In the above diagram, blue boxes represent physical machines or VMs and grey boxes represent containers. 
There are two physical/virual NetScaler instances present with IPs 10.0.0.1 and 10.0.0.2 and a NetScaler CPX (containerized NetScaler) with an IP 172.17.0.2.
To monitor stats and counters of these NetScaler instances, an exporter (172.17.0.3) is being run as a container. 
The exporter is able to get NetScaler stats such as http request rates, ssl encryption-decryption rate, total hits to a vserver, etc from the three NetScaler instances and send them to the Prometheus containter 172.17.0.4.
The Prometheus container then sends the stats acquired to Grafana which can plot them, set alarms, create heat maps, generate tables, etc as needed to analyse the NetScaler stats. 

   Details about setting up the exporter to work in an environment as given in the figure is provided in the following sections. A note on which NetScaler entities/metrics the exporter scrapes by default and how to modify it is also explained.

Usage:
---
The exporter can be run as a standalone python script, built into a container or run as a pod in Kubernetes.

<details>
<summary>Usage as a Python Script</summary>
<br>
To use the exporter as a python script, the ```prometheus_client``` package needs to be installed. This can be done using 
```
pip install prometheus_client
```
Now, the following command can be used to run the exporter as a python script;
```
nohup python exporter.py [flags] &
```
where the flags are:

flag             |    Description
-----------------|--------------------
--target-nsip    |Used to specify the &lt;IP:port&gt; of the Netscalers to be monitored
--port	        |Used to specify which port to bind the exporter to. Agents like Prometheus will need to scrape this port of the container to access stats being exported
-h               |Provides helper docs related to the exporter

The exporter can be setup as given in the diagram using;
```
nohup python exporter.py --target-nsip=10.0.0.1:80 --target-nsip=10.0.0.2:80 --target-nsip=172.17.0.2:80 --port 8888 &
```
This directs the exporter container to scrape the 10.0.0.1, 10.0.0.2, and 172.17.0.2, IPs on port 80, and the expose the stats it collects on port 8888. 
The user can then access the exported metrics directly thorugh port 8888 on the machine where the exporter is running, or Prometheus and Grafana can be setup to view the exported metrics though their GUI.
</details>



<details>
<summary>Usage as a Container</summary>
<br>

In order to use the exporter as a container, it needs to be built into a container. This can be done as follows; 
```
docker build -f Dockerfile -t ns-exporter:v1 ./
```
Once built, the general structure of the command to run the exporter is very similar to what was used while running it as a script:
```
docker run -dt -p [host-port:container-port] --name netscaler-exporter ns-exporter:v1 [flags]
```
To setup the exporter as given in the diagram, the following command can be used:
```
docker run -dt -p 8888:8888 --name netscaler-exporter ns-exporter:v1 --target-nsip=10.0.0.1:80 --target-nsip=10.0.0.2:80 --target-nsip=172.17.0.2:80 --port 8888
```
This directs the exporter container to scrape the 10.0.0.1, 10.0.0.2, and 172.17.0.2, IPs on port 80, and the expose the stats it collects on port 8888. 
The user can then access the exported metrics directly thorugh port 8888 on the machine where the exporter is running, or Prometheus and Grafana can be setup to view the exported metrics though their GUI.
</details>
