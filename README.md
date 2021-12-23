## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![TODO: Update the path with the name of your diagram](Images/diagram_filename.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the .yml file may be used to install only certain pieces of it, such as Filebeat.

<p>
<details>
  <summary>Ansible ELK Server Installation Playbook</summary>
  
<pre><code>---
- name: Configure ELK
  hosts: ELK
  remote_user: sysadmin
  become: True
  tasks:
  - name: use more memory
    sysctl:
      name: vm.max_map_count
      value: '262144'
      state: present
      reload: yes

  - name: docker.io
    apt:
      update_cache: yes
      name: docker.io
      state: present

  - name: Install pip3
    apt:
     force_apt_get: yes
     name: python3-pip
     state: present

  - name: install python module
    pip:
      name: docker
      state: present

  - name: elk container
    docker_container:
      name: elk
      image: sebp/elk:761
      state: started
      restart_policy: always
      published_ports:
        - 5601:5601
        - 9200:9200
        - 5044:5044
        
  - name: Enable Service docker on boot
    systemd:
      name: docker
      enabled: yes</code></pre>

  </details>
  </p>
  
  <p>
<details>
  <summary>DVWA Install File</summary>
  
  <pre><code>
  ---
- name: Config Web VM with Docker
  hosts: webservers
  become: true
  tasks:
  - name: docker.io
    apt:
      force_apt_get: yes
      update_cache: yes
      name: docker.io
      state: present

  - name: Install pip3
    apt:
      force_apt_get: yes
      name: python3-pip
      state: present

  - name: Install Docker python module
    pip:
      name: docker
      state: present

  - name: download and launch a docker web container
    docker_container:
      name: dvwa
      image: cyberxsecurity/dvwa
      state: started
      published_ports: 80:80

  - name: Enable docker service
    systemd:
      name: docker
      enabled: yes</code></pre>
  </details>
  </p>

<p>
<details>
  <summary>Filebeats Playbook</summary>
  
  <pre><code>---
- name: Installing and Launch Filebeat
  hosts: webservers
  become: yes
  tasks:
    # Use command module
  - name: Download filebeat .deb file
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb

    # Use command module
  - name: Install filebeat .deb
    command: dpkg -i filebeat-7.4.0-amd64.deb

    # Use copy module
  - name: Drop in filebeat.yml
    copy:
      src: /etc/ansible/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

    # Use command module
  - name: Enable and Configure System Module
    command: filebeat modules enable system

    # Use command module
  - name: Setup filebeat
    command: filebeat setup

    # Use command module
  - name: Start filebeat service
    command: service filebeat start

    # Use systemd module
  - name: Enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes</code></pre>
  </details>
  </p>
  
<p>
<details>
  <summary>Metricbeats Playbook</summary>
  
  <pre><code>---
- name: Install metric beat
  hosts: webservers
  become: true
  tasks:
    # Use command module
  - name: Download metricbeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb

    # Use command module
  - name: install metricbeat
    command: dpkg -i metricbeat-7.4.0-amd64.deb

    # Use copy module
  - name: drop in metricbeat config
    copy:
      src: /etc/ansible/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

    # Use command module
  - name: enable and configure docker module for metric beat
    command: metricbeat modules enable docker

    # Use command module
  - name: setup metric beat
    command: metricbeat setup

    # Use command module
  - name: start metric beat
    command: service metricbeat start

    # Use systemd module
  - name: Enable service metricbeat on boot
    systemd:
      name: metricbeat
      enabled: yes</code></pre>
  </details>
  </p>

This document contains the following details:
- Description of the Topologu
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting access to the network.
- Load Balancing contributes to the Availability aspect of security in regards to the CIA Triad.
- The advantage of a jump box is to give access to the user from a single node that can be secured and monitored as it is the origination point for launching Administrative Tasks.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the _____ and system _____.
- What does Filebeat watch for? Filebeat watches for any information in the file system which has been changed and when it has.
- What does Metricbeat record? Metricbeat records metric and statistical data from the operating system and from services running on the server and ships them to the output you specify.

The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

| Name     | Function | IP Address | Operating System |
|----------|----------|------------|------------------|
| Jump Box | Gateway  | 10.0.0.4   | Linux            |
| WEB-1    | SERVER   | 10.0.0.5   | Linux            |
| WEB-2    | SERVER   | 10.0.0.6   | Linux            |
| ELK-VM   | SERVER   | 10.2.0.4   | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Elk Server machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- Any Public IP through TCP 5601

Machines within the network can only be accessed by the Jumpbox Provisioner.
- Jump Box: 10.2.0.4 via SSH port 22
- Workstation Public IP via TCP 5601

A summary of the access policies in place can be found in the table below.

| Name          | Publicly Accessible     | Allowed IP Addresses             |
|---------------|-------------------------|----------------------------------|
|ELK-VM         | No                      | Public IP on SSH 22              |
| Jump Box      | No                      | Public IP on SSH 22              |
| Web-1         | No                      | 10.2.0.4    via SSH 22           |
| Web-2         | No                      | 10.2.0.4    via SSH 22           |
| Load Balancer | No                      | Public IP on HTTP 80             |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- You do not need to write custom scripts to automate your system or deployments. Ansible requires you to create tasks via playbook.yml and then easily deploy to all your machines designated by the host file. Ansible also figures out how to get each system to your desired state using these instructions.

The playbook implements the following tasks:
- **Machine groups and Remote User Specifications**
<pre><code>- name: Config ELK VM with Docker
  	  hosts: elk
  	  remote_user: sysadmin
  	  become: true
      tasks:</code></pre>

- **Increase System Memory**
 <pre><code>- name: Use more memory
    sysctl:
    name: vm.max_map_count
    value: '262144'
    state: present
    reload: yes</code></pre>

- **Install the following services**
  - Docker.io
  - Python3-pip
  - Docker elk container

- **Launching and Exposing the container with these published ports:**
  - _5601:5601_
  - _9200:9200_
  - _5044:5044_


The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![TODO: Update the path with the name of your screenshot of docker ps output](Images/docker_ps_output.png)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
| Name     | Function | IP Address | Operating System |
|----------|----------|------------|------------------|
| WEB-1    | SERVER   | 10.0.0.5   | Linux            |
| WEB-2    | SERVER   | 10.0.0.6   | Linux            |

We have installed the following Beats on these machines:
- FileBeats
- MetricBeats

These Beats allow us to collect the following information from each machine:
- Filebeats allow us to collect log events.
  - _ie: Files generated by Apache or MS Azure_

- Metricbeats allow us to collect metrics and statistics 
  - _ie: RAM Usage_ 

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the ELK Server Config file to /etc/ansinble.
- Update the hosts file to include Address of your webservers and ELK Private IP Address
- Update the filebeats-config.yml and the metricbeats-config.yml to designate the address of the ELK Server
  - output.elasticsearch: 
  - output.kibana: 

_TODO: Answer the following questions to fill in the blanks:_
- filebeat-playbook.yml
- metricbeat-playbook.yml
- Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on?
- /etc/ansible/hosts and add the private IP addresses under the [WEBSERVERS] section or [ELK Section] of the host
- Which URL do you navigate to in order to check that the ELK server is running?
- http:[Elk.VM.IP]:5601/app/kibana where the [Elk.VM.IP] is the Public address of the server

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._

<p>


  | Command | Purpose |
  | :---    | :---    | 
  | SSH -i [name of keygen file] (user@ipaddress) | Remote into your Jump Desktop |
  | SSH-Keygen | generate public and private key |
  | sudo apt install docker.io | Install Docker |
  | sudo service docker start | Start the Docker service |
  | sudo docker container list -a | show all containers services |
  | sudo docker start <container_name> | Start the container specified |
  | sudo docker attach <container_name> | Remote into the specified container |
  | ansible -m ping all | check the connection of ansible containers |
  | ansible-playbook <playbook.yml_file> | Run a playbook.yml file |
  | nano <name_of_playbook.yml> | Create an Ansible playbook |
  | sudo docker pull cyberxsecurity/ansible bash | run and create a docker image |
  | sudo docker ps -a | List all active/inactive containers |
  

 </p>
