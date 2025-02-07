# Infra Repository

Infrastructure as Code for Kubernetes cluster and application management using Ansible.

## Structure
- `ansible/`: Playbooks and inventory for cluster setup.

## Prerequisites
1. Install [Multipass](https://multipass.run/).
2. Install [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).

## Configuration
1. Create an `ansible.cfg` file in the `ansible/` directory with the following content:
   ```ini
   [defaults]
   inventory = ./inventory/inventory.ini
   host_key_checking = False
   ```

2. Create an `inventory.ini` file in the `ansible/` directory with the following content:
   ```ini
   [localhost]
   localhost ansible_connection=local
   ```

## Usage

3. Run the Ansible playbook with the `ANSIBLE_HOST_KEY_CHECKING` environment variable and `cluster_name` as an extra variable:
   ```sh
   ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook cluster-setup.yml -e cluster_name=prod-001
   ```

   - `ANSIBLE_HOST_KEY_CHECKING=False`: Disables SSH host key checking.
   - `-e cluster_name=prod-001`: Sets the `cluster_name` variable to `prod-001`.

   
## Playbook Overview
The playbook `cluster-setup.yml` performs the following tasks:

### Pre-tasks
- Fails if `cluster_name` is not defined.
- Cleans DHCP lease entries.
- Removes existing SSH known_hosts entries.

### Phase 1: VM Setup
- Launches VMs in parallel using Multipass.
- Waits for VMs to be in a running state.
- Sleeps for 30 seconds after VM launch.

### Phase 2: Network Setup
- Retrieves VM IPs.
- Updates the `/etc/hosts` file with the new host entries.

### Phase 3: SSH Setup
- Ensures the SSH key is set for the `ubuntu` user using Multipass.
- Waits for SSH to be available.
- Adds host keys to the known_hosts file.

### Phase 4: Package Installation
- Creates the keyrings directory.
- Adds the Kubernetes GPG key.
- Adds the Kubernetes repository.
- Installs necessary packages and Kubernetes tools.
- Sleeps for 30 seconds after package installation.

### Phase 5: Kubernetes Prerequisites
- Enables IP forwarding.
- Ensures the `br_netfilter` module is loaded.
- Ensures required sysctl parameters are set.
- Installs `containerd` and `runc`.
- Enables and starts the `kubelet` service.
- Configures `containerd` to use the systemd cgroup driver.
- Configures `kubelet` to use the systemd cgroup driver.
- Creates the `crictl` configuration file.

### Phase 6: Cluster Setup
- Initializes the Kubernetes master node.
- Sleeps for 30 seconds after `kubeadm init`.
- Sets up the kubeconfig for the master node.
- Copies the kubeconfig file from the master node to the local machine.
- Waits for the Kubernetes API server to be accessible.
- Installs Helm on the master node.
- Installs a pod network add-on (Cilium) using Helm.
- Waits for the pod network to be ready.
- Retrieves the join command from the master node.
- Joins worker nodes to the cluster.