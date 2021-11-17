## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![ELK Network Diagram](https://user-images.githubusercontent.com/64937715/142258869-d41a7c3b-b4a2-42d5-ab33-06ff1e516d53.jpg)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the ***install-elk.yml*** file may be used to install only certain pieces of it, such as Filebeat.

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly redundant, in addition to restricting buffer overflow to the network. In the case of a buffer overload, the load balancer allows users to maintain a connection to the DVWA that is accessible within the load balancer demarcatred by the three virtual machines: Web-1, Web-2, Web-3. If there is a buffer overload or if one of the virtual machines is down, the load balancer allows the user to maintain connection to the DVWA via another container. The JumpBoxProvisioner provides a layer of defense as it acts as the host machine that hosts the VMs within the network.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the logs and system statistics.
- Filebeat watches for logs.
- Metricbeat records system and service statistics.

The configuration details of each machine may be found below.

| Name               | Function  | IP Address | Operating System |
|--------------------|-----------|------------|------------------|
| JumpBoxProvisioner | Gateway   | 10.0.0.4   | Ubuntu 18.04     |
| Web-1              | DVWA      | 10.0.0.5   | Ubuntu 18.04     |
| Web-2              | DVWA      | 10.0.0.6   | Ubuntu 18.04     |
| Web-3              | DVWA      | 10.0.0.10  | Ubunut 18.04     |
| ELK-VM             | ELK Stack | 10.1.0.4   | Ubuntu 18.04     |


### Access Policies

In general, machines on an internal network are not exposed to the public internet. The current configuration in the Network Security Group, however, allows the JumpBoxProvisioner to be accessed via SSH Port 22 from a personal host machine IP located at: 
  - `72.205.80.152`

The ELK Stack Server is accessible via TCP/UDP (Port 5601) from the JumpBoxProvisioner with the personal host machine IP address of:
  - `72.205.80.152`

The ELK Stack Server can be reached only via SSH (Port 22) from the JumpBoxProvisioner using the private IP address:
  - `ssh cgdn@10.1.0.4`

Only the JumpBoxProvisioner machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
  - `72.205.80.152` - Private IP of personal host machine
  - SSH Port 22 with Dynamic Public IP Address:
    - `ssh cgdn@52.250.23.72`

Machines within the network can only be accessed by:
  - SSH Port 22
The only machine that is allowed access to the ELK virtual net is:
  - JumpBoxProvisioner
    - Private IP Address: 10.0.0.4
    - SSH Port 22

A summary of the access policies in place can be found in the table below.

| Name               | Publicly Accessible | Allowed IP Addresses |
|--------------------|---------------------|----------------------|
| JumpBoxProvisioner | Yes                 | 72.205.80.152        |
| Web-1              | No                  | 10.0.0.4             |
| Web-2              | No                  | 10.0.0.4             |
| Web-3              | No                  | 10.0.0.4             |
| ELK-VM             | No                  | 10.0.0.4             |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because configuration through Ansible allows us to repeat the process and apply the same configurations across multiple containers within the virtual network.

The Ansible playbook implements the following tasks:
- Use sysctl module
  - name: vm.max_map_count
  - value: "262144"
- Install docker.io
- Install Python3-pip
- Install Docker using pip
- Install ELK
  - Image: sebp/elk:761
  - Published Ports:
    - 5601:5601
    - 9200:9200
    - 5044:5044
- Enable docker service on restart

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

(ELK-Stack-Deployment/Diagrams/docker ps.png)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- Web-1 - 10.0.0.5
- Web-2 - 10.0.0.6
- Web-3 - 10.0.0.10

We have installed the following Beats on these machines:
- Filebeat
- Metricbeat

These Beats allow us to collect the following information from each machine:
- **Filebeat** records the system logs, such as syslogs, SSH login attempts, and sudo commands by user. For example, SSH login attempts logs displays the user(s) that have logged onto the the DVWA containers that are monitored by the ELK stack, revealing whom and how often the users have logged on within a given time period.
- **Metricbeat** records the CPU and memory usage of the 3 DVWA containers or the containers monitored by the ELK stack in general. For example, the Docker metrics are charted by timestamps measuring the CPU Usage, Network IO, and Memory Usage of each container monitored by the ELK Stack.

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:

- Copy the `install-elk.yml` file to the Ansible container.
- Update the `install-elk.yml` file to include the following:
```
---
  - name: Install and configure ELK stack container
    hosts: elk
    become: true
    tasks:

    - name: Use sysctl module
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes

    - name: Unstall apache httpd
      apt:
        name: apache2
        state: absent

    - name: Install docker.io
      apt:
        name: docker.io
        state: present

    - name: Install Python3-pip
      apt:
        name: python3-pip
        state: present

    - name: Install Docker using pip
      pip:
        name: docker
        state: present

    - name: Install ELK
      docker_container:
        name: elk
        image: sebp/elk:761
        restart_policy: always
        published_ports: 
          - 5601:5601
          - 9200:9200
          - 5044:5044

    - name: Enable docker service on restart
      systemd:
        name: docker
        enabled: yes
```
- Run the playbook with the following command:
```
ansible-playbook install-elk.yml 
```
- In order to make the Ansible run the playbook on specific machines, we must modify the `hosts` file. The following modifications were made on the `hosts` file for Ansible to determine which machines will be affected by the Filebeat and Metricbeat installations:
```
[webservers]
10.0.0.5 ansible_python_interpreter=/usr/bin/python3
10.0.0.6 ansible_python_interpreter=/usr/bin/python3
10.0.0.10 ansible_python_interpreter=/usr/bin/python3

[elk]
10.1.0.4 ansible_python_interpreter=/usr/bin/python3
```

- We must make modifications to the `ansible.cfg` file as well. We must modify the `remote_user` attribute to match the username that was used in the initial stages of setting up the Azure Virtual Network and Virtual Machines.

```
remote_user = cgdn
```

- From here, we must modify the `filebeat-config.yml` and `metricbeat-config.yml` files that will be in the directory:
  - /etc/ansible/files/filebeat-config.yml
  - /etc/ansible/files/metricbeat-confir.yml
- We will modify the settings under `Kibana` in accordance with the private IP address of the ELK server:
```
setup.kibana:
  host: "10.1.0.4:5601"
```
- These modifications will allow us to use our internet browser to access Kibana that is monitoring our containers (Web-1, Web-2, and Web-3):
- We can access Kibana using the following steps:
  - Obtain the public IP address of the ELK server listed on the Azure portal.
  - Using the following syntax, we can access Kibana through our browser:
    - `http://[ELK-server public IP]:5601/app/kibana`

- In order to check that the ELK server is running:
  - In your internet browser, navigate to:
    - http://[ELK server Public IP address]:5601/app/kibana
  - **Filebeat**: Observability > Logs > Add Log Data > System Logs > System Logs Dashboard
  - **Metricbeat**: Observability > Metrics > Add Metric Data > Docker Metrics > Docker Metrics Dashboard
