# ELK-EKS
ELK stack Kubernetes 

## Background

Connect ELK into EKS

---

## Source

https://github.com/ianmuge/elk-automation-challenge

https://github.com/kubernetes-sigs/aws-efs-csi-driver#installation

https://www.eksworkshop.com/beginner/190_efs/deploying-services/

## Architecture

ELK Stack Architecture setup and data flow

![https://cdn-images-1.medium.com/max/800/1*TXtAlgYFPYK1gubfGyfVig.png](https://cdn-images-1.medium.com/max/800/1*TXtAlgYFPYK1gubfGyfVig.png)


The setup was deployed on Google Kubernetes Engine on a standard cluster that was configured to allow autoscaling. The beats components-Metricbeat, Auditbeat and Filebeat- were deployed as Daemonsets to collect metrics and logs from the Kubernetes Cluster nodes and shipping them to Logstash. Logstash was deployed as deployment with a service load-balancing and exposing the beats ingest port and an API port. Logstash was configured with a pipeline to ingest the data from the beats daemons and ship it to Elasticsearch. Elasticsearch is deployed as a StatefulSet with 3 initial replicas ensuring a basic healthy cluster. Kibana interfaces with Elasticsearch to provide visualizations and dashboards.

## EFS CSI Driver on Kubernetes

```
helm repo add aws-efs-csi-driver https://kubernetes-sigs.github.io/aws-efs-csi-driver/ -n monitoring
helm repo update
helm upgrade --install aws-efs-csi-driver --namespace monitoring aws-efs-csi-driver/aws-efs-csi-driver
```

## Deployment
We bring everything together using Kustomize; it collates and formats all our manifests into one handy, easy to deploy manifest. **N.B** _This is not compulsory but simplifies our work especially when working with a CI/CD pipeline_ 
```
apiVersion: kustomize.config.k8s.io/v1
kind: Kustomization
resources:
  - setup.yml
  - elasticsearch.yml
  - kibana.yml
  - logstash.yml
  - apm-server.yml
  - beats/metricbeat.yml
  - beats/auditbeat.yml
  - beats/filebeat.yml
```
This then allows us to deploy using a single manifest.
```
kubectl kustomize . > compiled.yml
kubectl apply -f compiled.yml
```

## TEST
```
kubectl port-forward pods/[kibana-pod-name] -n monitoring
```