# Servas Application
Helm chart generated from Bolt.diy AI for the Servas self-hosted bookmark manager application.

References: 
- [Servas](https://servas.app/)
- [bolt.diy](https://github.com/stackblitz-labs/bolt.diy)
- [cursor.ai](https://www.cursor.com/)
- [Fleet](https://fleet.rancher.io/quickstart)

This will deploy the [Servas and Mariadb](https://github.com/kubernetes/examples/tree/master/guestbook/) application as
packaged as a Helm charts.
The solution will be deployed via Fleet which will be set up in a local Minikube cluster.

```
.
├── README.md
├── fleet-gitrepo-servas-mariadb.yaml
├── mariadb
│   ├── Chart.yaml
│   ├── fleet.yaml
│   ├── templates
│   │   ├── deployment.yaml
│   │   ├── pvc.yaml
│   │   └── service.yaml
│   └── values.yaml
└── servas
    ├── Chart.yaml
    ├── fleet.yaml
    ├── templates
    │   ├── configmap.yaml
    │   ├── deployment.yaml
    │   └── service.yaml
    └── values.yaml
```

```yaml
kind: GitRepo
apiVersion: fleet.cattle.io/v1alpha1
metadata:
  name: helm
  namespace: fleet-local
spec:
  repo: https://github.com/jamesrgregg/servas-app
  paths:
    - mariadb
    - servas
```
Configure Kubernetes cluster for Fleet deployments following the [documentation](https://fleet.rancher.io/quickstart).

`kubectl apply -f fleet-servas-mariadb.yaml`