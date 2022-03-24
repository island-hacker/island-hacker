## Project 1 - Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

### NETWORK DIAGRAM

https://github.com/island-hacker/island-hacker/blob/main/Project_1-ELK/Diagrams/Elk_Network_Diagram.png

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the `YML` file may be used to install only certain pieces of it, such as Filebeat.


  ### - install-elk.yml
  https://github.com/island-hacker/island-hacker/blob/main/Project_1-ELK/Ansible/install-elk.yml
  ### - filebeat-config.yml
  https://github.com/island-hacker/island-hacker/blob/main/Project_1-ELK/Ansible/filebeat-config.yml
  ### - metricbeat-playbook.yml
  https://github.com/island-hacker/island-hacker/blob/main/Project_1-ELK/Ansible/metricbeat-playbook.yml


---


### This document contains the following details:
    
  
    - Description of the Topology
    - Access Policies
    - ELK Configuration
        - Beats in Use
        - Machines Being Monitored
    - How to Use the Ansible Build

---
## Description of the Topology

    The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

    Load balancing ensures that the application will be highly `available`, in addition to restricting `access` to the network.

### -What aspect of security do load balancers protect? 

    The LB can take two times the traffic load hence it distributes traffic between the two servers and defends against denial of service (DDoS) attacks.

### - What is the advantage of a jump box?_

    The Jumpbox virtual machine has ansible installed therefore it helps automation of the creation & maintaince of multiple websevers and an Elk server. The Jumpbx is the only ssh gateway from outside of the internet.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the `files` and system `logs`.


### - What does Filebeat watch for?

    Filebeat watches for what files has been added, requested and how many times. EG (when a user trys to login to an elk server requesting a login page it will create a log file to be monitored later for any incorrect login attempts)

### - What does Metricbeat record?


    Metric beat will send computer usage (eg. memory usage, cpu usage) to elk. Elk contains the elastic search which works as search engine, logstash collects logs and kibana helps with the output and presentation of data.


The configuration details of each machine found below.

| Name        |     Function     | IP Address | Operating System |
|-------------|------------------|------------|------------------|
| Jump Box    |      Gateway     | 10.0.0.4   | Linux (Ubuntu)   |
| Webserver 1 |      Server      | 10.0.0.5   | Linux (Ubuntu)   |
| Webserver 2 |      Server      | 10.0.0.7   | Linux (Ubuntu)   |
| ELK Server  | Monitoring Server| 10.1.0.4   |Linux (Ubuntu)    |

___
## Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the `Jumpbox` machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:

### - Whitelisted IP addresses

    Home Public IP Address          58.179.191.66
  
Machines within the network can only be accessed by `Jumpbox`.

### - Which machine did you allow to access your ELK VM? What was its IP address?_

    Jumpbox VM Private IP Address     10.0.0.4
    Home PC Public IP Address         58.179.191.66


A summary of the access policies in place can be found in the table below.

| Name      | Publicly Accessible | Allowed IP Addresses |
|-----------|---------------------|----------------------|
| Jump Box  |       Yes           |   Host's Public IP   |
| Web1      |       No            |        10.0.0.5      |
| Web2      |       No            |        10.0.0.7      |
| Elk       |       No            |        Personal      |
#
## Elk Configuration

    
    Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
### - What is the main advantage of automating configuration with Ansible?_

    It does not require any person(s) to execute each installation and configuration steps as the person(s) would be waiting for some time depending on the installation therefore the person(s) would save time and use time for other task(s). Since the process can be repetative therefore human error(s) and speed can be overcome with automation. Ansible container can be used to run multiple installation on multiple servers from one remote location.


### - The playbook implements the following tasks:
      
    - Below task runs the playbook called install-elk.yml and it targets the ELK VM and true means to allow to be set at host level or it defaults to root.
                ---
                - name: install-elk.yml
                  hosts: elk
                  become: true
                  tasks:
      
      - Below task configures the target VM in this case the ELK VM to use more memory. The ELK container will not run without this setting. Module used in this case is the Ansible's sysctl module  which configures so settings automatically run if your VM has been restarted.
      
      Sysctl setting is also configured to 
                - name: more memory on vm
                  sysctl:
                    name: vm.max_map_count
                    value: 262144
      
      - Below task with the first 3 rows playbook runs apt install for Docker.io, present is a state of the ansible which maintains the state of the packages, like what should be the state of the package after the task is completed (present or absent) so in this case we need it to be present. Last task is to update the cache uppon the completion of the playbook.
                 -name: install docker.io
                  apt:
                    name: docker.io
                    state: present
                    update_cache: yes
       
      - Below task installs Python client for Docker which is required by Ansible to control the state of Docker containers, apt installer file name is python3-pip. As earlier we want the state to be present for the package.
                 - name: install python3
                   apt:
                     name: python3-pip
                     state: present

      - Below task installs Docker Engine which is used for running containers. Installer apt file name is called docker, as earlier we want the state to be present for the package.
                 - name: install docker python package
                   pip:
                     name: docker
                     state: present

      - Below task on 1st row playbook will attempt to configures a sebp/elk:761 container. sebp is the organisation that made the docker container, elk is the container and 761 is the version. Upon elk installation the elk will start, restart policy is set as always so elk virtual server starts on every restart because common cause of elk failing is because elk doesn't start when restarted. Next row is to configure following ports (5601 for Kibana, Elastic search 9200 , 5044 Logstash) so the services can be ready to be used. Container default behaviour set as compatibility which will ensure that the default values are used when the values are not explicitly specified by the user.

                - name: configure a sebp/elk:761 container
                  docker_container:
                  name: sebp_container
                  image:  sebp/elk:761
                  state: started
                  restart: yes
                  restart_policy: always
                  ports:
                    - "5601:5601"
                    - "9200:9200"
                    - "5044:5044"
                  container_default_behavior: compatibility

      - Task on the last row starts the elk docker from the systemd to make sure elk runs upon system restarts.

                - name: make sure docker service is started
                  systemd:
                    name: docker
                    enabled: yes
                    
      
    The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

    https://github.com/island-hacker/island-hacker/blob/main/Project_1-ELK/Diagrams/docker_ps_output.png

___


## Target Machines & Beats


### - This ELK server is configured to monitor the following machines:
          
    - Web 1 (10.0.0.5)
    - Web 2 (10.0.0.7)
    
### - We have installed the following Beats on these machines:
    
    - Filebeat
    - Metric Beat

### - These Beats allow us to collect the following information from each machine:

    - Filebeat assists to generate log files to Logstash and Elasticsearch which are two key features of an ELK, specifically filebeat logs information about the file system, including which files have changed, how many times they have been changed and when. Filebeat is often used to collect log files from very specific files, such as those generated by Apache, Microsoft Azure tools, the Nginx web server, and MySQL databases. Since Filebeat is built to collect data about specific files on remote machines, it must be installed on the VMs you want to monitor.

    - Metricbeat collects metrics and statistics on the installed system which then can send cpu usage and/or memory usage which can later be used to monitor the system's health.

---
## Using the Playbook
    
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

### - SSH into the control node and follow the steps below:
    
    
- Copy the _____ file to _____.

      Copy the `/etc/ansible/ansible.cfg` file from `the Ansible container to Elk VM in the /etc/ansible directory`.

### - Update the _____ file to include...
    
    Updating the `host /etc/ansible/hosts` fine to `include `webserver's Private IP addresses` & `ansible_python_interpreter=/usr/bin/python3` for the Webserver Group and `ELK VM's Private IP address 10.1.0.4 with ansible_python_interpreter=/usr/bin/python3`  for the ELK group.  

### - Run the playbook, and navigate to ____ to check that the installation worked as expected.

    Run the playbook, and navigate to
    `http://[your.ELK-VM.External.IP]:5601/app/kibana` using any browser to check that the installation worked as expected.

    Following webpage will load if successful.
    https://github.com/island-hacker/island-hacker/blob/main/Project_1-ELK/Diagrams/Completion.png

    You can also verify that `sebp/elk:761` container is running by using ssh command to ELK VM and running command `docker ps`.

    Photo attached as reference. 
    https://github.com/island-hacker/island-hacker/blob/main/Project_1-ELK/Diagrams/docker_ps_output.png


### - Which file is the playbook? Where do you copy it?_

    The filebeat configuration playbook `/etc/ansible/files/filebeat-config.yml` gets copied as `/etc/filebeat/filebeat.yml` in the webserver VM's.

### - Which file do you update to make Ansible run the playbook on a specific machine? 

    Updating the host file `/etc/ansible/hosts` with `webserver's Private IP addresses & ansible_python_interpreter=/usr/bin/python3` for the Webserver Group and `ELK VM's Private IP address 10.1.0.4 with ansible_python_interpreter=/usr/bin/python3`  for the ELK group.

      eg.
        [webservers]
        `10.0.0.5 ansible_python_interpreter=/usr/bin/python3`
        `10.0.0.7 ansible_python_interpreter=/usr/bin/python3`

      
       [elk]
       `10.1.0.4 ansible_python_interpreter=/usr/bin/python3`


### - How do I specify which machine to install the ELK server on versus which to install Filebeat on?_



      Updating the host file `/etc/ansible/hosts` with`ELK VM's Private IP address 10.1.0.4 with ansible_python_interpreter=/usr/bin/python3` for the ELK group so the Elk

      eg.
        [webservers]
        `10.0.0.5 ansible_python_interpreter=/usr/bin/python3`
        `10.0.0.7 ansible_python_interpreter=/usr/bin/python3`
              
       [elk]
       `10.1.0.4 ansible_python_interpreter=/usr/bin/python3`
               

        The URL to navigate to in order to check that the ELK server is running is `http://[your.ELK-VM.External.IP]:5601/app/kibana`.
        https://github.com/island-hacker/island-hacker/blob/main/Project_1-ELK/README/Images/Completion.jpg.bmp


_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._

specific commands the user will need to run to download the playbook, 

update the files, 

etc._

### - Filebeat Playbook
    
      filebeat-playbook.yml
    ---
      - name: Playbook to install & configure Filebeat
        hosts: webservers
        become: true
        tasks:

        - name: Download the Filebeat installation file
          command:
          cmd: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb
          creates: filebeat-7.6.1-amd64.deb

        - name: Install Filebeat
          command: sudo dpkg -i filebeat-7.6.1-amd64.deb

        - name: Copy the Filebeat config file to the ElkVM "ubuntuVM"
          copy:
            src: /etc/ansible/roles/file/filebeat-config.yml
            dest: /etc/filebeat/filebeat.yml

        - name: Enable Filebeat modules
          command: filebeat modules enable system

        - name: Filebeat setup
          command: filebeat setup

        - name: Start Filebeat
          command: service filebeat start

        - name: Enable Filebeat service on boot
          systemd:
            name: filebeat
            enabled: yes

---

### METRICBEAT PLAYBOOK


         ---
    - name: Create a play to install Metricbeat
      hosts: webservers
      become: true
      tasks:

      - name: download metricbeat installation file
        command:
          cmd: curl -L -O https://artifacts.elastic.co/downloads/ beats/metricbeat/metricbeat-7.6.1-amd64.deb
          creates: metricbeat-7.6.1-amd64.deb

      - name: install metricbeat
        command: sudo dpkg -i metricbeat-7.6.1-amd64.deb

      - name: copy metricbeat config file to the ELKVM "ubuntuNM"
        copy:
          src: /etc/ansible/files/metricbeat-config.yml
          dest: /etc/metricbeat/metricbeat.yml

      - name: enable metricbeat modules docker
        command: metricbeat modules enable docker

      - name: setup metricbeat
        command: metricbeat setup

      - name: enable metricbeat service on boot
        systemd:
          name: metricbeat
          state: started
          enabled: yes

---


