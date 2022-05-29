---
layout: post
title:  "Virtualized Malware Analysis Environment"
categories: [DFIR, Malware_Analysis]
tags: [Malware Analysis]
---

## **Background** <br/>  

### Cuckoo Sandbox Project

<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">
Cuckoo Sandbox project is an open-sourced tool that automates dynamic malware analysis. The project was built using python language which makes the installment process easier to install across all platform. Cuckoo also supports malware analysis on Windows, OSX, Linux, and Android. Cuckoo, also, have a well-defined structure that makes it easy to customize. This feature has allowed analysts across the world to add custom modules and plugins that capture certain artifacts to result the best outcome. Lastly, Cuckoo have a great community support for the project which beyond the scope of this blog but could be found on this github repo. [Cuckoo Sandbox Community](https://github.com/cuckoosandbox/community)
</span>

### SIFT Workstation

<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">
SIFT Workstation is a powerful forensics framework that contains most of the open-source tools used by industry-level analysts. SIFT workstation comes in the form of an appliance and could be ran as a virtual machine. Reducing the overhead of installing and configuring each tool is one of its greatest advantage.
</span>

### Ansible
<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">
Ansible is an open-source software and powerful tools that could be used for various aspects. Ansible mainly know for four overall functionality:  
 Application Deployment (Like Fabric)  
 Provisioning (Like Cobbler or JuJu)  
 Configuration Management (Like Chef or Puppet)  
 Multi-tier Orchestrion (Like Chef-Metal)  

### Vagrant

<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">
Vagrant is a provisioning platform from hashicorp that is used to spins up and maintain virtual machines in different hypervisor providers such as AWS, VMware and virtualbox.
open keyboard shortcuts file.
</span>

## **Introduction**

<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">
Configuring and setting up different services manually is both time-consuming and labor-intensive. In this project,  I tried to implement an automated solution for a malware analyst to deploy a sandboxed environment using different technology. Starting by Ansible scripts that deploy SANS workstation to a VMware vCenter host then deployment Cuckoo sandbox software along with all its required dependencies using ansible. After that, I used Vagrant to build a virtual machine (In my case, I 'll be using Windows 10, however, there is plenty of supported OS), to be used for my malware analysis.    </span>

##    **Constructed Topology**

<img src="https://raw.githubusercontent.com/sh1dow3r/sh1dow3r.github.io/master/_posts/img/Sandbox/Virtualized_Malware_Analysis_Environment.png"/>

<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">
As depicted in the topology we have a computer that communicated with VMware vCenter to deploy the SIFT workstation. After that, we configure the workstation with Cuckoo project using Ansible. Lastly, we provision VirtualBox using vagrant to spawn up a virtual machine to be used for malware analysis.
</span>

#  **Install/Setup**

<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">  
Before diving into the project let's take a quick look at the structure of the code:   </span>
<img src="https://raw.githubusercontent.com/sh1dow3r/sh1dow3r.github.io/master/_posts/img/Sandbox/Sandboxer_Dir_Structure.png"/>

<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">
As you can here, we have a bunch of files, starting from the top, we have:  
`SIFT-Cuckoo-Playbook.yml`: which contains the overall tasks(roles) related to Cuckoo project setup.  
`SIFT-Deploy.yml`: which contains the vCenter deployment of SIFT workstation.  
`inventory.ini`: contains the variables used to communicate with the other node such IP, user and password of the client machine  
`requirements.sh`: has all the dependencies used by this project.  
`roles`: has the different subtask of the project  
`roles\SIFT-Cuckoo-Sandbox\*`: subtask to install and configure Cuckoo sandbox project  
`vars.yml`: a centralized place to hold all the variables of the project.  <br/> 
To install the project, nvaigete th efolling github repo and clone it.  
`git clone https://github.com/sh1dow3r/SandBoxer`  
`cd SandBoxer`  
`./requirements.sh` to install all the dependencies  
Edit `vars.yml` accordingly  
The rest will follows in the next section.
</span>

##  **How does it work?**
### - SIFT Workstation Deployment
<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">
After we install the dependencies for the project we start with the first step, which is deploying our workstation to vCenter. To do so, we will edit the `vars.yml` section to  have the required information we need to deploy the vm.  
Here is a quick example of what my file looks
<img src="https://raw.githubusercontent.com/sh1dow3r/sh1dow3r.github.io/master/_posts/img/Sandbox/vars_content.png"/>
`vars.yml` file contains a lot of variables related to the vCenter API. After making sure, you have all the variables set.  Additionally, make sure you have the SIFT ova file in the correct path. Now we will run the the playbook `SIFT-Deploy.yml` to deploy SIFT workstation using the follwoing command.
`ansible-playbook SIFT-Deploy.yml`  
<img src="https://raw.githubusercontent.com/sh1dow3r/sh1dow3r.github.io/master/_posts/img/Sandbox/SIFT_Deploment_vCenter.png"/>

<img src="https://raw.githubusercontent.com/sh1dow3r/sh1dow3r.github.io/master/_posts/img/Sandbox/Ansible_output_after_vcenter_deployment.png"/>
<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">  
As you can see, in the previous screenshot the ansible playbook is completed and the vm is deployed on the vCenter server. (Note: you may discard the warnings while running the playbook).

### - Cuckoo Project Deployment

<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">  
Now that we have deployed the virtual machine, let's starts the the Cuckoo deployment. Starting by adding the required variables in the `vars.yml`. After you have inputted the necessary vars, you can run the Cuckoo ansible playbook as follows:  
`ansible-playbook SIFT-Cuckoo-Sandbox.yml -i inventory.ini`
After issuing the command we wait for a few minutes for the whole setup to be done then we hop onto the workstation to confirm the installment as seen in the screenshot below
</span>
<img src="https://raw.githubusercontent.com/sh1dow3r/sh1dow3r.github.io/master/_posts/img/Sandbox/cuckoo_after_installing.png"/> 

<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">
After we confirmed all the configuration is done, we'll navigate to `home/cuckoo/vagrant_files` which has the windows10 virtual machine we're going to use for malware analysis. There should be a bash script called `vagrant_script.sh` that is going to pull a windows 10 image and set it up with Virtualbox then take a stable snapshot.
<img src="https://raw.githubusercontent.com/sh1dow3r/sh1dow3r.github.io/master/_posts/img/Sandbox/After_vagrant.png"/> 

<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">
After windows10 is up and running make sure it's snapshotted and there you go, you have your Cuckoo Sandbox up and ruining! :
</span>
<img src="https://raw.githubusercontent.com/sh1dow3r/sh1dow3r.github.io/master/_posts/img/Sandbox/Cuckoo_up_and_running.png"/> 


## Conclusion

<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">
Building Malware analysis environment manually requires time, resource and can bee overwhelming to setup. In this project, I proposed a solution of an automated process
to setup you're own malware environment using DevOps tools like Ansible and Vagrant. </span >

# References


[Ansible docs](https://docs.ansible.com/ansible/latest/modules/lineinfile_module.html)

[Vagrant-Virtualbox provider](https://www.vagrantup.com/docs/virtualbox/)

[Cuckoo Ansible example](https://github.com/fyhertz/ansible-role-cuckoo)
