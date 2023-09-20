[![published](https://static.production.devnetcloud.com/codeexchange/assets/images/devnet-published.svg)](https://developer.cisco.com/codeexchange/github/repo/ucs-compute-solutions/FlexPod-UCSM-VMware)

# FlexPod Converged Infrastructure setup using Ansible for FlexPod Datacenter with Cisco UCSM M6, VMware vSphere 8, and NetApp ONTAP 9.12.1

This repository for FlexPod contains Ansible playbooks to configure Cisco Nexus, Cisco UCS Intersight, Cisco MDS, NetApp ONTAP, VMware ESXi, and VMware vCenter. This repository can be used for setting up Cisco devices, NetApp ONTAP, and VMware ESXi and vCenter as covered in the following Cisco Validated Design (CVD): https://www.cisco.com/c/en/us/td/docs/unified_computing/ucs/UCS_CVDs/flexpod_iac_e2d_deploy.html.

The CVD lays out the complete process for configuring the FlexPod using Ansible. Since these playbooks are intended to save time in setting up a working FlexPod, a complete FlexPod as shown below is needed to execute the playbooks. Various simulators could be used to partially test individual playbooks.

![block-diagram](https://github.com/ucs-compute-solutions/FlexPod-UCSM-VMware/blob/main/ReadmePics/Main-Topology.jpg)  

# Set up the execution environment

To execute various ansible playbooks, a linux based system will need to be setup as described in the CVD with the packages listed at the following pages:

- Cisco UCS: https://galaxy.ansible.com/cisco/ucs
- Cisco NxOS: https://galaxy.ansible.com/cisco/nxos
- NetApp ONTAP: https://galaxy.ansible.com/netapp/ontap
- VMware: https://galaxy.ansible.com/community/vmware

# How to execute these playbooks?

![block-diagram](https://github.com/ucs-compute-solutions/FlexPod-UCSM-VMware/blob/main/ReadmePics/Ansible-Order.jpg)

Because a number of manual tasks need to be executed between running the Ansible playbooks, the CVD document should be used as a guide for running the playbooks. Commentary is included in the variable files to guide filling in those values.

In this version of the FlexPod setup, FC boot with FC-NVMe is configured by default in the variable files, but iSCSI boot and NVMe-TCP can also be used.
The steps for setting up a FlexPod with FC boot with FC-NVMe and NFS storage protocols are:

1.  Create a directory and clone the repository from Github with "git clone https://github.com/ucs-compute-solutions/FlexPod-UCSM-VMware.git".
2.  Fill in the variable files according to the CVD.
3.  Follow the manual steps in the CVD to set up the Nexus switches on the network and ssh into each switch.
4.  Execute the Nexus playbook with "ansible-playbook ./Setup_Nexus.yml -i inventory" to setup the Nexus switches.
5.  Follow the manual steps in the CVD to add timezone information to the Nexus switches.
6.  Follow the manual steps in the CVD to get the NetApp storage cluster on the network.
7.  Execute the NetApp storage playbook with "ansible-playbook -i inventory Setup_ONTAP.yml -t ontap_config_part_1".
8.  Query the Infra-SVM FC and FC-NVMe LIF WWPNs or iSCSI IQN and add to the "all.yml" file.
9.  Follow the manual steps in the CVD to set up the UCS Fabric Interconnects on the network.
10.  Follow the manual steps in the CVD to set up the FC unified ports on the Fabric Interconnects.
11.  Execute the UCS playbook with "ansible-playbook ./SetupUCS.yml -i inventory" to setup the Cisco UCS Service Profile Templates, Policies, and Pools.
12.  Follow the manual steps in the CVD to create UCS service profiles for three or more VMware ESXi management hosts.
13.  Query the ESXi host IQNs or WWPNs from the service profiles and add to the "all.yml" file.
14.  If configuring Fibre Channel, follow the manual steps in the CVD to set up the MDS switches on the network and ssh into each switch.
15.  If configuring Fibre Channel, execute the MDS playbook with "ansible-playbook ./Setup_MDS.yml -i inventory".
16.  If configuring Fibre Channel, follow the manual steps in the CVD to add timezone information to the MDS switches. 
17.  Execute the NetApp storage playbook with "ansible-playbook -i inventory Setup_ONTAP.yml -t ontap_config_part_2" to create and map the ESXi boot LUNs.
18.  Follow the manual steps in the CVD to install VMware ESXi on the three (or more) host servers and assign IPs to those servers.
19.  Execute the ESXi playbook with "ansible-playbook ./Setup_ESXi.yml -i inventory" to setup the ESXi hosts.
20.  Bring a vCenter into the environment by either installing vCenter on the first ESXi host according to the CVD, copying it in, or establishing L3 routing to it.
21.  Setup the vCenter and add the three or more ESXi hosts to it by executing "ansible-playbook ./Setup_vCenter.yml -i inventory".
22.  Follow the manual steps in the CVD to complete setting up vCenter and the ESXi hosts.
23.  Execute the NetApp storage playbook with "ansible-playbook -i inventory Setup_ONTAP.yml -t ontap_config_part_3" to setup NVMe-TCP and finalize ONTAP Storage.
24.  Execute the manual steps in the CVD to complete the NVMe-TCP setup.
25.  Follow the steps in the CVD to install the ONTAP Tools VM via Ansible.
26.  Follow the steps in the CVD to intsall the SnapCenter VMware Plug-In via Ansible.
27.  Follow the steps in the CVD to install Active IQ Unified Manager via Ansible.
28.  Follow the manual steps in the CVD to finish setting up ONTAP tools, the SnapCenter Plug-in, and AIQUM.
29.  Follow the manual steps in the CVD to setup Cisco Intersight Assist and Cisco Nexus Dashboard Fabric Controller (NDFC) SAN.

The Ansible playbooks and CVD are structured in a way that a Fibre Channel Boot, Fibre Channel Boot with FC-NVMe, iSCSI Boot or iSCSI Boot with NVMe-TCP FlexPod or combination configurations can be setup by adjusting the variables. Also, the playbooks can be used to setup the following topology utilizing Cisco Nexus switches that support SAN Switching (93180YC-FX, 93360YC-FX2, or 9336C-FX2-E) for both LAN and SAN switching and 100G FCoE Uplinks from the FIs to the switches.

![block-diagram](https://github.com/ucs-compute-solutions/FlexPod-UCSM-VMware/blob/main/ReadmePics/NexusSAN-Topology.jpg)

