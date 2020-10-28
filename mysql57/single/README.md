
# Simple stateful Mysql DB deployment on Platform9 Managed Kubernetes 4.5


MySQL is one of the most popular database servers. Here we are going to bootstrap a single Mysql 5.7 database instance server on top of Platform9 Managed kubernetes 4.5.2 with helm. 

# Prerequisites
1. Dynamic persistent volume provisioning with CSI driver with [Rook](https://github.com/KoolKubernetes/csi/tree/master/rook/) or a suitable CSI volume provisioner.
2. [Platform9 Managed Kubernetes 4.x](https://docs.platform9.com/release-notes/)

