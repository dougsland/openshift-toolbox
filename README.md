# OpenShift Enterprise 3.11  

Some personal notes, feel free to send patches, comments or fixes

## Operational System
* RHEL 7 (Lastest tested 7.6)
* SELinux enabled (default)
* Firewall enabled (default)
* Network Manager enabled (default)

## Requirements
 * Red Hat subscription (RHEL + OpenShift)
 * At least two physcal or virtual machines (nested kvm), one for master and other for node. 

## Installation notes/flow
* During the RHEL installation
  * [Use a single / partition]((https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/sect-disk-partitioning-setup-x86#sect-custom-partitioning-x86)), no need to split in /home, /var, /usr
  * Make sure you set [Desired Capacity](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/sect-disk-partitioning-setup-x86#sect-custom-partitioning-x86) less than total available to have free space.
  ```Example, from 80G disk, desire capacity 50G```

* Set [DNS or /etc/hosts (static ip address)](https://docs.openshift.com/container-platform/3.11/getting_started/install_openshift.html#install-prerequisites) for master and nodes

* Register master/node to Red Hat [subscription system and enable the channels](https://docs.openshift.com/container-platform/3.11/getting_started/install_openshift.html#attach-subscription)
  ```
  # subscription-manager register
  # subscription-manager refresh
  
  # subscription-manager list --available
  # subscription-manager attach --pool=<pool_id>
  
  # subscription-manager repos --enable="rhel-7-server-rpms"
    --enable="rhel-7-server-extras-rpms" \
    --enable="rhel-7-server-ose-3.11-rpms" \
    --enable="rhel-7-server-ansible-2.6-rpms"
  ```

* [Install the OpenShift Container Platform Package](https://docs.openshift.com/container-platform/3.11/getting_started/install_openshift.html#install-package)
  ```
  # yum -y install wget git net-tools bind-utils iptables-services bridge-utils bash-completion kexec-tools sos psacct
  # yum -y update
  # yum -y install openshift-ansible
  # yum -y install docker
  ```

* Start/Enable docker service in master and nodes
  ```
  # systemctl start docker 
  # systemctl enable docker`
  ```

* [Set up Password-less SSH Access](https://docs.openshift.com/container-platform/3.11/getting_started/install_openshift.html#set-up-password-less-ssh)

   In master generate the ssh keys and distribute to nodes
   ```
   # ssh-keygen
   
   # for host in master.openshift.example.com \
        node.openshift.example.com; \
        do ssh-copy-id -i ~/.ssh/id_rsa.pub $host; \
        done
   ```
  
* **BEFORE** running the Ansible Playbook, **install skopeo package in MASTER and NODES**
  ```
  # yum install skopeo -y
  ```
* Create the ansible inventory file
  ```
  # vi /etc/ansible/hosts
  ```
  *Example: https://github.com/dougsland/openshift-toolbox/blob/master/ansible/hosts*

* Run the Installation Playbooks

  Check prerequisites:
   ```
    # cd /usr/share/ansible/openshift-ansible
    # ansible-playbook -i /etc/ansible/hosts playbooks/prerequisites.yml
   ```
   
   Install:
   ```
    # cd /usr/share/ansible/openshift-ansible
    # ansible-playbook -i /etc/ansible/hosts playbooks/deploy_cluster.yml 
   ```
   *For verbose install add -vvv*

# After installation
* Login via web
 https://master.mydomain.com:8443/

* Console:
  ```
  # oc login -u system:admin
  # oc describe node
  # oc get pods
  ```
  
* Create the first project:
  ```
  # oc new-project my-super-app
  ```
  
* Create a pod/persistance volume/resource from YAML
  ```
  # oc create -f path/filname.yaml
  ```

* remove a pod/persistance volume/resource from YAML
  ```
  # oc remove -f path/filname.yaml
  ```

* Additional commands:
  ```
  # oc get events  (show events, good for research some details/failures)
  
  # oc get pods (list pods)
  # oc rsh <pod-name> (rsh into pod)
  
  # oc get pv  (Pesistance Volume)
  # oc get pvc (Persistance Volume Claim)
  ```
  
  A PersistentVolume (PV) object represents a piece of existing networked storage in the cluster that has been provisioned by an administrator. It is a resource in the cluster just like a node is a cluster resource. PVs are volume plug-ins like Volumes, but have a lifecycle independent of any individual pod that uses the PV. PV objects capture the details of the implementation of the storage, be that NFS, iSCSI, or a cloud-provider-specific storage system.
  
  A PersistentVolumeClaim (PVC) object represents a request for storage by a user. It is similar to a pod in that pods consume node resources and PVCs consume PV resources. For example, pods can request specific levels of resources (e.g., CPU and memory), while PVCs can request specific storage capacity and access modes (e.g, they can be mounted once read/write or many times read-only).

# Uninstall
  * Uninstall openshift completely
    ```
    # ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/adhoc/uninstall.yml
    ```
  
---
# Troubleshooting 3.11 installation

* No schedulable nodes found matching node selector for Cluster Monitoring Operator - 'node-role.kubernetes.io/infra=true'

  **Possible Solution**:  
In the [nodes] Section, add `openshift_node_group_name="node-config-infra"`

*  Hosts:    master.foobar.com, node1.foobar.com  
     Play:     Configure nodes  
     Task:     Check that node image is present  
     Message:  non-zero return code

   **Possible Solution**:  
  Check installation logs but make sure the **docker** daemon is running in **master** and **nodes**

*    Hosts:    master.foobar.com  
     Play:     Approve any pending CSR requests from inventory nodes  
     Task:     Approve node certificates when bootstrapping  
     Message:  Could not find csr for nodes: node1.foobar.com

     **Possible Solution**:  
   Not clear yet, reboot is recommended
   
* "The following packages have pending transactions: atomic-openshift-clients-x86_64"

     **Possible Solution**:  
   ```
   # yum-complete-transaction --cleanup-only
   ```

# Resources
[Getting started - version 3.11](https://docs.openshift.com/container-platform/3.11/getting_started/)
