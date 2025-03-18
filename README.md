# Servas Application
Helm chart for the Servas self-hosted bookmark manager application.



## Project Description:
This will deploy the [Servas and Mariadb](https://github.com/kubernetes/examples/tree/master/guestbook/) application to Kubernetes single-cluster (minikube).  
The original docker-compose file was used as the base for converting to helm charts using several approaches including AI generated helm charts.
The solution will be deployed via Fleet which will be set up in a local Minikube cluster.
The service defined for the `servas-service` is set as a `NodePort` and will require port-forward command (manual) to allow access to to the internal frontend servas-service endpoint.
The busybox container will be used as a side car to allow for app key generation for the servas app.

```
.
├── README.md
├── busybox
│   ├── chart.yaml
│   ├── fleet.yaml
│   ├── templates
│   │   └── deployment.yaml
│   └── values.yaml
├── fleet-servas-mariadb.yaml
├── mariadb
│   ├── chart.yaml
│   ├── fleet.yaml
│   ├── templates
│   │   ├── deployment.yaml
│   │   ├── pvc.yaml
│   │   └── service.yaml
│   └── values.yaml
└── servas
    ├── chart.yaml
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
    - busybox
```
Configure Kubernetes cluster for Fleet deployments following the [documentation](https://fleet.rancher.io/quickstart).

`kubectl apply -f fleet-servas-mariadb.yaml`

### Key Learning
The [CoPilot](https://github.com/copilot/c/5018258b-fc27-4342-bd8c-73b6d95ccc54) generated helm charts were a good starter but were created with both servas and mariadb containers running inside the same pod.  This did not follow the required separation of frontend from backend services and would not be a solution that followed best practices.  Deploying this on a minikube cluster gave insights into other considerations that included the need for a SSL certificate on the frontend, TLS between the frontend -> backend, and container security (running as non-root).

[Bolt.diy](https://github.com/stackblitz-labs/bolt.diy) generated the helm charts in a similar way as helm templating (manual).  Deployment with Fleet did not work - job was created but no containers were created.  **Need to check namespace for this deployment**  

[Kompose](https://kompose.io) quickly generated the helm charts but did not include fleet.yaml or charts.yaml.  This code was not tested but is found [here](https://github.com/jamesrgregg/kompose2helm).  I used this repo and the original docker-compose to spin up the solution locally just to understand how it works.  I could not get it to work properly and the [Issues](https://github.com/jamesrgregg/kompose2helm/blob/main/servas-app/README.md) encountered helped with additional security considerations.

Resorted to using Helm templates for this repo and manually updated as needed (e.g. `helm create <chart name>`)


References: 
- [Servas](https://servas.app/)
- [Servas Source Code](https://github.com/beromir/Servas)
- [bolt.diy](https://github.com/stackblitz-labs/bolt.diy)
- [cursor.ai](https://www.cursor.com/)
- [Fleet](https://fleet.rancher.io/quickstart)
- [Fleet Uninstall - Cleanup](https://fleet.rancher.io/uninstall)
- [Kompose](https://kompose.io)
- [Reusing Existing Kubernetes Secrets in Helm Templates](https://blog.cloudcover.ch/posts/reusing-existing-kubernetes-secrets-in-helm-templates/)
- [A Hands-On Guide to Kubernetes Jobs ](https://medium.com/@muppedaanvesh/a-hand-on-guide-to-kubernetes-jobs-%EF%B8%8F-aa2edbb061ea)
- [Connecting Applications with Services](https://kubernetes.io/docs/tutorials/services/connect-applications-service/)

## Video Training / Tutorials
 - [Kubernetes Master Class - Rancher Continuous Delivery with Fleet](https://www.youtube.com/watch?v=lNeX_PxnzLM&t=15s)
 - [Using Rancher Fleet and GitHub Actions for GitOps Releases](https://www.youtube.com/watch?v=kagSBoInW6g&t=556s)
 - [DevOps Toolkit: Rancher Fleet](https://gist.github.com/vfarcic/eabe08e8e147fb2ce51afc520efc0cef)