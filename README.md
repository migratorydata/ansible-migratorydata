## ansible-migratorydata

## Introduction

Welcome to the documentation for `ansible-migratorydata`. This guide will walk you through the process of setting up a cluster of servers with MigratoryData Push Server using Ansible, a powerful automation tool that can configure systems, deploy software, and orchestrate more advanced IT tasks.

Our `ansible/install.yaml` script simplifies the process of installing a MigratoryData Push Server cluster. With this script, you can easily automate the setup of your cluster, saving you time and ensuring consistency across your servers.

In addition to the installation, we also provide a `ansible/monitor.yaml` script for setting up monitoring for your cluster. This script leverages Prometheus and Grafana, two leading open-source tools for monitoring and visualizing metrics. With this script, you can keep a close eye on your cluster's statistics and logs, helping you to maintain the health and performance of your servers.

Finally, we understand that keeping your servers up-to-date is crucial for security and performance. That's why we've included an `ansible/update.yaml` script that automates the process of updating the MigratoryData Push Server on all instances. With this script, you can ensure that your cluster is always running the latest version of MigratoryData Push Server.

Whether you're setting up a new cluster, monitoring its performance, or updating your servers, `ansible-migratorydata` provides the tools you need to automate these tasks.

> Tested with Ansible 2.11.5, Debian 12 and Ubuntu 22.04.

## Prerequisites

Ensure that you have installed the following tools:

  - [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
  - one or more Debian 12 or Ubuntu 22.04 machines

## Install Ansible on MacOS

You can install Ansible on Mac OS using the following command:

```bash
brew install ansible
```

## Configure the deployment

Clone the `ansible-migratorydata` repository:

```bash
git clone https://github.com/migratorydata/ansible-migratorydata
cd ansible-migratorydata
```

Update if necessary the configuration files from the `deploy/configs` directory. See the [Configuration](https://migratorydata.com/docs/server/configuration/) guide for more details. If you developed custom extensions, add them to the the `deploy/extensions` directory.

## SSH keys

In order to establish a secure connection to the virtual machines, Ansible requires both public and private SSH keys. These keys can be generated on your local machine using the ssh-keygen command. Once generated, the path to the private key can be specified in the `hosts.ini` file using the `ansible_ssh_private_key_file` variable. Follow the steps below to generate a new SSH key pair:

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

Executing this command will create a new RSA key pair with a key size of 4096 bits. The keys will be stored in the `.ssh` directory of your home folder. The public key will be saved as `~/.ssh/id_rsa.pub` and the private key as `~/.ssh/id_rsa`.

## Installing Ansible Collections

The provided commands are used to install necessary Ansible collections. Ansible collections are a distribution format for Ansible content that can include playbooks, roles, modules, and plugins.

```bash
# This command installs the community.general collection, which includes many of the most commonly used modules and other resources for Ansible.
ansible-galaxy collection install community.general

# This collection is designed to interact with Prometheus, a powerful open-source monitoring and alerting toolkit.
ansible-galaxy collection install prometheus.prometheus

# This collection provides modules to interact with Grafana, a popular open-source platform for visualizing metrics, which complements Prometheus.
ansible-galaxy collection install grafana.grafana
```

By running these commands, you ensure that your Ansible environment has the necessary collections to manage and monitor your infrastructure effectively.

## Configure infrastructure

1. Creating the `artifacts/hosts.ini` file: 
- This file is crucial for Ansible as it defines the hosts (or machines) that Ansible will manage. The `hosts.ini` file is created in the `artifacts` directory and contains two groups: `migratorydata` and `monitor`.

- The `migratorydata` group lists the machines that will run the MigratoryData Push Server. Each machine is defined by its public IP address, the Ansible user, the path to the SSH private key, the private IP of the node, and the IP of the monitoring machine(optional).
- The `monitor` group lists the machines that will run the monitoring tools (Prometheus and Grafana). Each machine is defined by its IP address and the Ansible user.
  - While monitoring is a valuable tool for maintaining the health and performance of your MigratoryData cluster, it is not a mandatory requirement for the installation and operation of the cluster. The `monitor` group in the `hosts.ini` file is optional and the MigratoryData cluster can be installed and run without it. Also the `monitor_ip` is optional and can be removed from the `migratorydata` group.


Here is an example of a cluster with three machines and a monitoring machine in the `hosts.ini` file:

```ini
[migratorydata]
machine1_ip ansible_user=admin ansible_ssh_private_key_file=ssh_private_key node_private_ip=machine1_private_ip monitor_ip=monitor_machine_ip
machine2_ip ansible_user=admin ansible_ssh_private_key_file=ssh_private_key node_private_ip=machine2_private_ip monitor_ip=monitor_machine_ip
machine3_ip ansible_user=admin ansible_ssh_private_key_file=ssh_private_key node_private_ip=machine3_private_ip monitor_ip=monitor_machine_ip

[monitor]
monitor_machine_ip ansible_user=admin ansible_become=True
```

2. Creating the `clustermembers` file: 
- This file is used by the `ansible/install.yaml` playbook to update MigratoryData Push Server configuration file with the nodes addresses that  are part of the cluster. The `clustermembers` file is created in the `artifacts` directory using the `ansible-playbook ansible/cluster-members.yaml -i artifacts/hosts.ini` command. This command runs the `cluster-members.yaml` playbook with the hosts.ini file as the inventory, generating the `clustermembers` file with the necessary information.

To create the `artifacts/clustermembers` file, run the following command:

```bash
ansible-playbook ansible/cluster-members.yaml -i artifacts/hosts.ini
```

## Install MigratoryData Push server

By running these commands, you initiate the installation of the MigratoryData Push Server on your infrastructure. The `ansible/install.yaml` playbook automates the installation process, ensuring a consistent setup across all your servers.

```bash
# used to disable SSH host key checking
export ANSIBLE_HOST_KEY_CHECKING=False

ansible-playbook ansible/install.yaml -i artifacts/hosts.ini
```

## Monitoring setup

If the `monitor` groups in the `hosts.ini` is configured and the variable `monitor_ip` is set for each server in `migratorydata` group, then you can install grafana and prometheus on the machines using the following command:

```bash
# For mac you need tar and gnu-tar
brew install gnu-tar

# Fix for MAC OS error `ERROR! A worker was found in a dead state`
export OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES
# used to disable SSH host key checking
export ANSIBLE_HOST_KEY_CHECKING=False

ansible-playbook ansible/monitor.yaml -i artifacts/hosts.ini
```

## Verifying deployment

After you have successfully installed the MigratoryData Push Server on your cluster, it's important to verify that the deployment was successful. Here's how you can do that:

- You can access the MigratoryData cluster by navigating to the public IP address of any of the virtual machines in your web browser. The MigratoryData Push Server runs on port 8800 by default, so you would access it at `http://machine_public_ip:8800`. If the server is running correctly, you should see the MigratoryData welcome page.

- You can also verify the deployment by SSH-ing into the virtual machines. You can do this using the command `ssh admin@machine_public_ip` or `ssh -i ssh_private_key admin@machine_public_ip` if you need to specify a private key. Once logged in, you can check the status of the MigratoryData service and inspect the logs to ensure everything is running as expected.

```bash
sudo su
# Check the status of the MigratoryData service
systemctl status migratorydata

# Inspect the logs
nano /var/log/migratorydata/all/out.log
```

## Update MigratoryData Push server

The `package_url` and `package_name` variables in the `ansible/vars.yaml` file should be updated to point to the new version of the MigratoryData Push Server. The `package_url` is the URL where the new version can be downloaded, and the `package_name` is the name of the package file.

Update MigratoryData Server running the following commands: 

```bash
# used to disable SSH host key checking
export ANSIBLE_HOST_KEY_CHECKING=False

ansible-playbook ansible/update.yaml -i artifacts/hosts.ini
```

The `ansible/update.yaml` playbook contains tasks that update the MigratoryData Push Server on the machines specified in the `hosts.ini` inventory file. The playbook is designed to sequentially update the MigratoryData Push Server on each server in your infrastructure. The playbook operates on one host to ensure service availability during the update process. The tasks executed on each host include stopping the MigratoryData service, downloading the new MigratoryData Push Server package, updating the package, and restarting the service. After updating each server, Ansible pauses for a period of time before moving to the next server, allowing the updated server to fully restart and rejoin the cluster before the next server is updated.

## Build realtime apps

Use any of the MigratoryData's [client APIs](https://migratorydata.com/docs/client-api/) to develop real-time applications for communication with this MigratoryData cluster.
