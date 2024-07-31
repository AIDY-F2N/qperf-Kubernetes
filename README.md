# Deployment of qperf Tool on Kubernetes for Bandwidth and Latency Measurement

<div align="center">
    <img src="figures/1_IconsAll_Hori.png" alt="qperf">
</div>


## Description

This repository provides instructions and configurations to deploy a qperf tool on a Kubernetes cluster for measuring TCP bandwidth and latency between every pair of workers. The qperf tool will be deployed in a pod, and the measurements will be collected periodically and stored in a JSON file (data.json).


## Contributor

- Massinissa AIT ABA, massinissa.ait-aba@davidson.fr

## Pre-requisite
- Kubernetes cluster configured and accessible via kubectl.
- Basic understanding of Kubernetes pods, deployments, and services.


## Deployment Steps


1.  Step 1: Clone this repository

```bash
git clone https://github.com/AIDY-F2N/qperf-Kubernetes.git
cd qperf-kubernetes
```
    
2. Deploy the qperf pod using the provided deployment.yaml:

```bash
kubectl apply -f Deployment.yaml
```

This YAML file defines a Kubernetes Deployment that creates a qperfall pod which will run the measurements. The qperfall pod contains script will start qperf tests between each pair of workers every 60 seconds and store the results in data.json located within the pod.



## Viewing Results

- Access the pod and view the results directly:


```bash
kubectl exec -it $(kubectl get pods --all-namespaces | awk '/qperfall/{print $2}') -n qperf -- /bin/bash
cat data.json
```
- Alternatively, copy the results file from the pod to your local machine:

```bash
kubectl cp -n qperf $(kubectl get pods -n qperf | awk '/qperfall/{print $1}'):data.json ./data.json
```

- View the results stored in data.json:
```bash
cat data.json
```

```yaml
{
  {
    "2024-06-28T13:02:38.358264": {
        "node1": {
            "node2": {
                "bw": 63.2,
                "latency": 154.0
            }
        },
        "node2": {
            "node1": {
                "bw": 60.8,
                "latency": 105.0
            }
        }
    },
    "2024-06-28T13:03:48.689863": {
        "node1": {
            "node2": {
                "bw": 60.4,
                "latency": 144.0
            }
        },
        "node2": {
            "node1": {
                "bw": 63.6,
                "latency": 116.0
            }
        }
    },
    "2024-06-28T13:04:58.895727": {
        "node1": {
            "node2": {
                "bw": 61.3,
                "latency": 143.0
            }
        },
        "node2": {
            "node1": {
                "bw": 61.2,
                "latency": 155.0
            }
        }
    }
}
}
```

In this data.json example:

- Each timestamp represents a measurement point.
- node1 and node2 represent two worker nodes.
- Under each timestamp, bandwidth (bw) measurement is provided in Mb/s and latency (latency) measurement is provided in milliseconds, in both directions between node1 and node2.
This structure allows for tracking and analysis of network performance metrics over time between the specified nodes. Adjustments can be made to the actual data structure based on specific needs and additional nodes involved in the measurements.

## Bandwidth and Latency Measurement Overview:
The qperfall pod deployed within the Kubernetes environment conducts continuous measurements of network performance metrics, specifically focusing on:

1- Bandwidth (bw): Bandwidth refers to the data transfer rate between two nodes in the network. qperfall calculates and records the bandwidth values between different pairs of nodes within the Kubernetes cluster. Typically measured in bits per second (bps) or higher units like kilobits per second (Kbps), megabits per second (Mbps), or even gigabits per second (Gbps) for higher-speed networks.

2- Latency: Latency represents the time delay between sending and receiving data packets over the network. qperfall measures latency between nodes to assess the responsiveness and efficiency of network communication. Usually measured in milliseconds (ms), latency values indicate how quickly data travels between nodes.


Operational Workflow:

1- Deployment: qperfall deploys qperf as a DaemonSet within the qperf namespace, ensuring it runs on every worker in the cluster.

2- Execution: qperfall executes the following command for each pair of workers within the Kubernetes cluster:

```bash
qperf -ip <port_number> <destination_ip> tcp_bw tcp_lat &
```

## Accessing the qperf Pod for Debugging

If you need to troubleshoot the qperf pod in your Kubernetes cluster, you can access it using the following command:

```bash
kubectl exec -it $(kubectl get pods --all-namespaces | awk '/qperfall/{print $2}') -n qperf -- /bin/bash
```
Once inside the pod, you can view the logs by running:

```bash
cat std.log
```
This will display the logs of the qperf pod. The std.log file contains detailed logging messages regarding the deployment and operation of the qperf-daemonset. These messages include information about successful creation and deletion of the DaemonSet, along with specifics on exact and normalized measurements of bandwidth (bw) and latency (latency) between different worker instances (workers). Each log entry is timestamped and structured to provide accurate tracking of operations performed by the qperf pod within the Kubernetes environment.


## Cleaning Up
To remove the qperf deployment from your cluster, run:

```bash
kubectl delete -f Deployment.yaml
```
This will delete the qperf pod and associated resources from your Kubernetes cluster. It is essential to clean up the qperf pod after collecting your data to avoid continuous usage of bandwidth and latency, as qperf tests run every 60 seconds.
