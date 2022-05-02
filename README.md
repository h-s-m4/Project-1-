# Project-1- 
---

## ELK-Stack-Project
**Automated ELK Stack Deployment**
 
The files in this repository were used to configure the network depicted below.

![vNet Diagram](https://github.com/hibo-m/Project-1-/blob/0dd7876ea6dcb8059acff1cedf68c5be99fb7f54/hmk%2013.PNG)
 
These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the Azure Cloud Environment file may be used to install only certain pieces of it, such as Filebeat and Metricbeat.

  - [install-elk.yml](https://github.com/hibo-m/Project-1-/blob/9b1d15408e49b66f4cea67af76594050cc59dad0/configfiles/elk.yml)
  - [filebeat-config.yml](https://github.com/hibo-m/Project-1-/blob/9b1d15408e49b66f4cea67af76594050cc59dad0/configfiles/filebeat-configuration.yml)
  - [filebeat-playbook.yml](https://github.com/hibo-m/Project-1-/blob/9b1d15408e49b66f4cea67af76594050cc59dad0/configfiles/filebeat-playbook.yml)
  - [metricbeat-config.yml](https://github.com/hibo-m/Project-1-/blob/9b1d15408e49b66f4cea67af76594050cc59dad0/configfiles/metricbeat-configuration.yml)
  - [metricbeat-playbook.yml](https://github.com/hibo-m/Project-1-/blob/9b1d15408e49b66f4cea67af76594050cc59dad0/configfiles/metricbeat-playbook.yml)
 
This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
- Beats in Use
- Machines Being Monitored
- How to Use the Ansible Build
 
### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the Damn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting inbound access to the network.

>Load balancing ensures that the application will be highly reliable, in addition to restricting access to the network.

>Load Balancers protect against DoS attacks. The advantage of the jump box is that it restricts access to one administrator. Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the actual machines and system logs.

>Filebeat helps generate and organize log files to send to Logstash and Elasticsearch. Specifically, it logs information about the file system, including which files have changed and when.

>Metricbeat is a lightweight shipper that you can install on your servers to periodically collect metrics from the operating system and from services running on the server. Metricbeat takes the metrics and statistics that it collects and ships them to the output that you specify, such as Elasticsearch or Logstash.

The configuration details of each machine may be found below.
 
| Name     | Function | IP Address | Operating System |
|----------|----------|------------|------------------|
| Jump-Box-Provisioner | Gateway  | 20.239.61.30 , 10.0.0.8   | Linux            |
| Web-1        |webserver    | 10.0.0.9     | Linux            |
| Web-2        |webserver    | 10.0.0.10     | Linux            |
| ELKServer    |Kibana       | 10.2.0.4 ,  20.222.15.169    | Linux            |
| RedTeam-LB|Load Balancer| 52.175.127.15| DVWA            |


### Access Policies
 
The machines on the internal network are not exposed to the public Internet.

Only the Jump Box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:

>20.239.61.30 - my personal machine.

>Machines within the network can only be accessed by accessing the DVWA container in the Jump Box VM.

>The only machines that can access the ELK server are 20.239.61.30 - my personal machine, and the Jump Box VM at IP 10.0.0.8 through a peering connection.
 
A summary of the access policies in place can be found in the table below.
 
| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump-Box-Provisioner | Yes      | 99.245.444.55
| ELKServer      | Yes            | 99.245.444.55 :5601|
| DVWA 1   | No                  |  10.0.0.1-254        |
| DVWA 2   | No                  |  10.0.0.1-254        |


 
---


### ELK Configuration
 
Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...

>You don’t need to install any other software or firewall ports on the client systems you want to automate. You also don’t have to set up a separate management structure.
					     
>The playbook implements the following tasks:

>Install Docker
	
>Download Image
	
>Configure container
	
>Create playbook to install container with docker and Filebeat and Metricbeat.
	
>Run playbook to launch the container

<details>
<summary> <b> Click here to view ELK Configuration. </b> </summary>

---
 
We will configure an ELK server within virtual network. Specifically,
 
- Deployed a new VM on our virtual network.
- Created an Ansible play to install and configure an ELK instance.
- Restricted access to the new server.

#### Deployed a new VM on our virtual network. 
 
1. Create a new vNet located in the same resource group we have been using. 
- Make sure this vNet is located in a new region and not the same region as our other VM's, which region we select is not important as long as it's a different US region than our other resources, we can also leave the rest of the settings at default.
	

![Create vNet](https://github.com/hibo-m/Project-1-/blob/main/elk%20project%2013.PNG)  

2. Create a Peer connection between our vNets. This will allow traffic to pass between our vNets and regions. This peer connection will make both a connection from our first vNet to our second vNet and a reverse connection from our second vNet back to our first vNet. This will allow traffic to pass in both directions.
- Navigate to `Virtual Network` in the Azure Portal.
- Select our new vNet to view it's details.
- Under `Settings` on the left side, select `Peerings`.
- Click the + Add button to create a new Peering.
- A unique name of the connection from our new vNet to our old vNet such as depicted example below.
- Choose our original RedTeam vNet in the dropdown labeled `Virtual Network`.
- Leave all other settings at their defaults.
 
![PeeringsELKtoRed](https://github.com/hibo-m/Project-1-/blob/main/elk%20peerings.PNG) 
 
![PeeringsRedtoELK](https://github.com/hibo-m/Project-1-/blob/main/redteam%20peerings.PNG)  

3. Create a new Ubuntu VM in our virtual network with the following configurations:
- The VM must have a public IP address.
- The VM must be added to the new region in which we created our new vNet. We want to make sure we select our new vNEt and allow a new basic Security Group to be created for this VM.
- The VM must use the same SSH keys as our WebserverVM's. This should be the ssh keys that were created on the Ansible container that's running on our jump box.
- After creating the new VM in Azure, verify that it works as expected by connecting via SSH from the Ansible container on our jump box VM.

   - ```bash
        ssh sysadmin@<jump-box-provisioner>
     ``` 
   - ```bash
        sudo docker container list -a
     ``` 
   - ```bash
        sudo docker start sweet_grothendieck && sudo docker attach sweet_grothendieck
     ``` 
 
![connect_on_newVM](https://github.com/hibo-m/Project-1-/blob/main/sudo%20container%20list.PNG)  
 
 

#### Created an Ansible play to install and configure an ELK instance.

In this step, we have to:
- Add our new VM to the Ansible hosts file.
- Create a new Ansible playbook to use for our new ELK virtual machine.
- From our Ansible container, add the new VM to Ansible's hosts file.
   - RUN `nano /etc/ansible/hosts` and put our IP with `ansible_python_interpreter=/usr/bin/python3`

![hosts file editing](https://github.com/hibo-m/Project-1-/blob/main/ansible.1.PNG)  

- In the below play, representing the header of the YAML file, I defined the title of my playbook based on the playbook's main goal by setting the keyword 'name:' to: "Configure Elk VM with Docker". Next, I defined the user account for the SSH connection, by setting the keyword 'remote_user:' to "sysadmin" then activated privilege escalation by setting the keyword 'become:' to "true". 
 
 The playbook implements the following tasks:

---
    
```yaml
---
- name: Configure Elk VM with Docker
  hosts: elkservers
  remote_user: azureserver
  become: true
  tasks:
```

In this play, the ansible package manager module is tasked with installing  'pip3', a version of the 'pip installer' which is a standard package manager used to install and maintain packages for Python.
The keyword 'force_apt_get:' is set to "yes" to force usage of apt-get instead of aptitude. The keyword 'state:' is set to "present" to verify that the package is installed.

 
```yaml
     # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present
```
	
In this play the pip installer is used to install docker and also verify afterwards that docker is installed ('state: present').

```yaml
      # Use apt module
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present
```
	
```yaml
     # Use pip module
    - name: Install Docker python module
      pip:
        name: docker
        state: present
```	
	
In this play, the ansible sysctl module configures the target virtual machine (i.e., the Elk server VM) to use more memory. On newer version of Elasticsearch, the max virtual memory areas is likely to be too low by default (ie., 65530) and will result in the following error: "elasticsearch | max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]", thus requiring the increase of vm.max_map_count to at least 262144 using the sysctl module (keyword 'value:' set to "262144"). The keyword 'state:' is set to "present" to verify that the change was applied. The sysctl command is used to modify Linux kernel variables at runtime, to apply the changes to the virtual memory variables, the new variables need to be reloaded so the keyword 'reload:' is set to "yes" (this is also necessary in case the VM has been restarted).

```yaml
      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes
```

In this play, the ansible docker_container module is used to download and launch our Elk container. The container is pulled from the docker hub repository. The keyword 'image:' is set with the value "sebp/elk:761", "sebp" is the creator of the container (i.e., Sebastien Pujadas). "elk" is the container and "761" is the version of the container. The keyword 'state:' is set to "started" to start the container upon creation. The keyword 'restart_policy:' is set to "always" and will ensure that the container restarts if we restart our web vm. Without it, we will have to restart our container when we restart the machine.
The keyword 'published_ports:' is set with the 3 ports that are used by our Elastic stack configuration, i.e., "5601" is the port used by Kibana, "9200" is the port used by Elasticsearch for requests by default and "5400" is the default port Logstash listens on for incoming Beats connections (we will go over the Beats we installed in the following section "Target Machines & Beats").

```yaml
      # Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044
```

In this play, the ansible systemd module is used to start docker on boot, setting the keyword 'enabled:' to "yes".

```yaml
      # Use systemd module
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes
```
![Install_elk_yml](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Install_elk_yml.png)

Now we can start launching and exposing the container by run

```bash
ansible-playbook elk.yml
```

The following screenshot displays the result of running `elk.yml`

![Docker ELKResult output](https://github.com/hibo-m/Project-1-/blob/main/ansible%20playbook.PNG)

SSH to our container: ```ssh azureuser@10.2.0.4``` and RUN ```sudo docker ps```

The following screenshot displays the result of running `docker ps` after successfully configuring the Elastic Stack instance.

![Docker InstallELK output](https://github.com/hibo-m/Project-1-/blob/main/docker%20ps.PNG)

Logging into the Elk server and manually launch the ELK container with: 

```bash
sudo docker start elk
```
then ```curl http://localhost:5601/app/kibana``` does return HTML.

The following screenshot displays the result of running `curl` after start ELK container

![Docker curl output](https://github.com/hibo-m/Project-1-/blob/main/elk%20curl%20kibana.PNG)

#### Restricted access to the new server.
	
This step is to restrict access to the ELK VM using Azure's network security groups (NSGs). We need to add public IP address to a whitelist, just as we did when clearing access to jump box.

Go to Network Security Group to config our host IP to Kibana as follow

![Docker InboundSecRules output](https://github.com/hibo-m/Project-1-/blob/main/access%20from%20home.PNG)

Then try to access web browser to http://<your.ELK-VM.External.IP>:5601/app/kibana 
 
![Access_Kibana](https://github.com/h-s-m4/Project-1-/blob/main/kibana%20home%20page.PNG)

</details>

---

### Target Machines & Beats
This ELK server is configured to monitor the following machines:

- Web-1 (DVWA 1) | 10.0.0.9
- Web-2 (DVWA 2) | 10.0.0.10

I have installed the following Beats on these machines:

- Filebeat
- Metricbeat

<details>
<summary> <b> Click here to view Target Machines & Beats. </b> </summary>

---

	
These Beats allow us to collect the following information from each machine:

`Filebeat`: Filebeat detects changes to the filesystem. I use it to collect system logs and more specifically, I use it to detect SSH login attempts and failed sudo escalations.

We will create a [filebeat-config.yml](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Ansible/filebeat-config.yml) and [metricbeat-config.yml](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Ansible/metricbeat-config.yml) configuration files, after which we will create the Ansible playbook files for both of them.

Once we have this file on our Ansible container, edit it as specified:
- The username is elastic and the password is changeme.
- Scroll to line #1106 and replace the IP address with the IP address of our ELK machine.
output.elasticsearch:
hosts: ["10.1.0.4:9200"]
username: "elastic"
password: "changeme"
- Scroll to line #1806 and replace the IP address with the IP address of our ELK machine.
	setup.kibana:
host: "10.1.0.4:5601"
- Save both files filebeat-config.yml and metricbeat-config.yml into `/etc/ansible/files/`

![files_FMconfig](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/files_FMconfig.png) 
 
 
Next, create a new playbook that installs Filebeat & Metricbeat, and then create a playbook file, `filebeat-playbook.yml` & `metricbeat-playbook.yml`

RUN `nano filebeat-playbook.yml` to enable the filebeat service on boot by Filebeat playbook template below:

```yaml
---
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
      src: /etc/ansible/files/filebeat-config.yml
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

```

![Filebeat_playbook](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Filebeat_playbook.png) 
 
- RUN `ansible-playbook filebeat-playbook.yml`

![Filebeat_playbook_result](https://github.com/h-s-m4/Project-1-/blob/main/filbeat%20playbook.PNG)  

Verify that our playbook is completed by navigate back to the Filebeat installation page on the ELK server GUI
	
![Filebeat_playbook_verify1](https://github.com/h-s-m4/Project-1-/blob/main/filebeat%20kibana.PNG)
		
	
`Metricbeat`: Metricbeat detects changes in system metrics, such as CPU usage and memory usage.

RUN `nano metricbeat-playbook.yml` to enable the metricbeat service on boot by Metricbeat playbook template below:

```yaml
---
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
      src: /etc/ansible/files/metricbeat-config.yml
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
```

![Metricbeat_playbook](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Metricbeat_playbook.png)  
 
- RUN `ansible-playbook metricbeat-playbook.yml`

![Metricbeat_playbook_result](https://github.com/h-s-m4/Project-1-/blob/main/metricbeat-playbook.PNG)  

 
</details>

---
 
### Using the Playbook

Next, I want to verify that `filebeat` and `metricbeat` are actually collecting the data they are supposed to and that my deployment is fully functioning.

---
<details>
<summary> <b> Click here to view Using the Playbook. </b> </summary>

In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned:

SSH into the control node and follow the steps below:

Copy the configuration and playbook YAML files, to the /etc/ansible/files folder using:
cp ./Resources/filebeat-config.yml /etc/ansible/files/filebeat-config.yml
cp ./Resources/filebeat-play.yml /etc/ansible/files/filebeat-play.yml
cp ./Resources/metricbeat-config.yml /etc/ansible/files/metricbeat-config.yml
cp ./Resources/metricbeat-play.yml /etc/ansible/files/metricbeat-play.yml
cp ./Resources/install-ELK.yml /etc/ansible/files/install-ELK.yml

Update the filebeat-config and metricbeat-config files each to point towards the ELK server IP address, username and password. (10.2.0.4, elastic & changeme, respectively, for both config files)

Update the /etc/ansible/hosts file with two separate sections for the private server IP addresses of the ELK server and the webservers on the internal network. (see above for IP addresses) configure two sections: [webservers] [elkservers]

Run the filebeat and metricbeat playbooks, and ssh into each webserver (10.0.0.9, 10.0.0.10) and run "curl localhost/setup.php" to verify installation worked.
enter: ssh azureuser@server-ip-address (replace "server-ip-address" with above IP addresses

You should get an HTML code response on-screen. Next, navigate to the webservers via the load balancer's public IP address (91.474.29.175/setup.php) to check that the installation worked as expected.

Run the install-ELK playbook and navigate to the ELK server's Kibana webpage using the ELK server's public IP address at the following URL (http://20.27.168:5601/app/kibana#/home) to check that the installation worked as expected. You should see the Kibana homepage.

If everything runs as specified above, congratulations on your success!
	

![metricbeat kibana](https://github.com/h-s-m4/Project-1-/blob/main/metricbeat%20kibana.PNG)

  .

   








