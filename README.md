# k8s-vps-cluster-lab
Documentation of setting up Kubernetes (K3s) on a multi-node cluster, from server hardening to installations.
```
kubernetes-on-vps/
│
├── README.md
├── architecture/
│   ├── cluster-architecture.md
│   ├── network-design.md
│   └── diagrams/
│
├── docs/
│   ├── 01-vps-setup.md
│   ├── 02-server-hardening.md
│   ├── 03-install-container-runtime.md
│   ├── 04-install-kubernetes.md
│   ├── 05-create-cluster.md
│   ├── 06-networking.md
│   ├── 07-storage.md
│   └── troubleshooting.md
│
├── scripts/
│   ├── init-server.sh
│   ├── install-containerd.sh
│   ├── install-kubernetes.sh
│   └── join-node.sh
│
├── manifests/
│   ├── nginx-deployment.yaml
│   ├── ingress.yaml
│   └── storage-class.yaml
│
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
│
├── ansible/
│   ├── inventory
│   ├── playbook.yml
│   └── roles/
│
└── images/
    └── architecture-diagram.png
```
