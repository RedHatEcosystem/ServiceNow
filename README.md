# ServiceNow
Setup for using Ansible and ServiceNow with bidirectional communication -- for service catalog items to launch AAP2 job templates


# Installing IntegrationHub to your ServiceNow instance.  
An administrator with access to the ServiceNow customer service portal will need to install the plugin, IntegrationHub.  There are four levels of IntegrationHub from low to high: Starter, Standard, Professional, Enterprise. Only the top two include Ansible Spoke. In the instances section, select "Activate Plugin" and search for the plugin (IntegrationHub Professional or Enterprise), schedule a time, then submit.


<img src="https://raw.githubusercontent.com/RedHatEcosystem/ServiceNow/main/install_inthub.png">

# Installing Ansible Spoke to your ServiceNow instance.  
Red Hat TPP instances should already have IntegrationHub Enterprise installed.  IntegrationHub Enterprise or Professional are prerequisites for Ansible Spoke.  If the next step doesn't work, contact your admin about getting one of the higher level IntegrationHub packages installed to your ServiceNow instance.

<img src="https://raw.githubusercontent.com/RedHatEcosystem/ServiceNow/main/install_spoke.png">

