# ServiceNow Automation and Service Catalog
Setup for using Ansible and ServiceNow with bidirectional communication -- for service catalog items to launch AAP2 job templates

# Requirements

* Automation Controller Instance that is exposed to the internet.
* Valid certificates for Controller (Check out this role that requests certificated and configures Nginx to use them)
* Admin access to the ServiceNow Instance

# Configuring your Automation Controller instance

## Creating an application token for SNow -> Controller communication

1. Log on to Automation Controller
2. On the left panel under `Administration`, click on `Applications`

![Automation Controller](/images/controller-1.png)

3.In the main panel, click on the `Add` Button

![Automation Controller](/images/controller-2.png)

4. Enter the following information for the new Application Token: 

| Key                      | Value                                                                                      | Note                               |
|--------------------------|--------------------------------------------------------------------------------------------|------------------------------------|
| Name                     | ServiceNow                                                                                 |                                    |
| Description              | Token for ServiceNow to controller connectivity                                            |                                    |
| Organization             | Default                                                                                    | Choose your preferred Organization |
| Authorization grant type | Authorization Code                                                                         |                                    |
| Redirect URIs            | https://`<YOUR_SNOW_INSTANCE>`.service-now.com/api/sn_ansible_spoke/ansible_oauth_redirect | Replace `<YOUR_SNOW_INSTANCE>`     |
| Client type              | Confidential                                                                           l   |                                    |

Then click `Save`

![Automation Controller](/images/controller-3.png)

5. The `Application Information` windows will pop-up with your `Client ID` and 
`Client Secret`, copy both values to your text editor for now. 

![Automation Controller](/images/controller-4.png)


> **Tip**
>
> This is the **only** time you will be able to see/copy the `Client Secret` Value, if you lose it you will have to restart the process with a new application.


## Creating a credential type and credential for Controller -> SNow communication
TODO: Add screenshots for cred type

Input Configuration:
```yml
---
fields:
  - id: username
    type: string
    label: Service Now Username
  - id: password
    type: string
    label: Service Now Password
    secret: true
  - id: instance
    type: string
    label: Service Now Instance
required:
  - username
  - password
  - instance
```

Injector Configuration:
```yml
---
env:
  SN_INSTANCE: 'https://{{ instance }}.service-now.com'
  SN_PASSWORD: '{{password}}'
  SN_USERNAME: '{{username}}'
extra_vars:
  snow_instance: '{{ instance }}'
  snow_password: '{{ password }}'
  snow_username: '{{ username }}'

```

# Configuring Your ServiceNow Instance

## Installing IntegrationHub to your ServiceNow instance.  

An administrator with access to the ServiceNow customer service portal will need to install the plugin, IntegrationHub.  There are four levels of IntegrationHub from low to high: Starter, Standard, Professional, Enterprise. Only the top two include Ansible Spoke. In the instances section, select "Activate Plugin" and search for the plugin `ServiceNow IntegrationHub Enterprise Pack Installer`, schedule a time, then submit. 

![Install Integration Hub](/images/install_inthub.png)

## Installing Ansible Spoke to your ServiceNow instance.  
Red Hat TPP instances should already have IntegrationHub Enterprise installed.  IntegrationHub Enterprise or Professional are prerequisites for Ansible Spoke.  If the next step doesn't work, contact your admin about getting one of the higher level IntegrationHub packages installed to your ServiceNow instance.

![Install Integration Hub](/images/install_spoke.png)

## Configuring Ansible Spoke:

1. Log on to ServiceNow using your `admin` account
2. On the left panel type in `Flow` in the `Filter navigator` and Click on `Flow Designer` Under `Process Automation`. The `Flow Designer` will open in a new tab.

![Service Now](/images/snow-1.png)



3. In the `Flow Designer`, click on the `Connections` tab

![Service Now](/images/snow-2.png)

4. Under the `Connections` tab, there will be a connection for `Ansible Tower Alias` with the identifier `sn_ansible_spoke`. In the box for that connection, click on the `Add Connection` button

![Service Now](/images/snow-3.png)

5. The `Create Connection` box will pop up. Enter the following information:

| Key                       | Value                                                                                      | Note                                    |
|---------------------------|--------------------------------------------------------------------------------------------|-----------------------------------------|
| Connection Name           | AnsibleTowerAlias                                                                          |                                         |
| Connection URL            | https://`<YOUR_ANSIBLE_CONTROLLER_URL>`                                                    | Replace `<YOUR_ANSIBLE_CONTROLLER_URL>` |
| Credential Name           | Controller Spoke Credentials                                                               |                                         |
| Application Registry Name | Controller                                                                                 |                                         |
| OAuth Client ID           | `<CLIENT_ID_FROM_CONTROLLER_APPLICATION_CREATED_EARLIER`                                   | Replace with your Client ID             |
| OAuth Client Secret       | `<CLIENT_SECRET_FROM_CONTROLLER_APPLICATION_CREATED_EARLIER>`                              | Replace with your Client Secret         |
| Oauth Entity Profile Name | Ansible Entity Profile                                                                     |                                         |
| Oauth Entity Scope        | write                                                                                      |                                         |
| Authorization URL         | https://`<YOUR_ANSIBLE_CONTROLLER_URL>`.com/api/o/authorize/                               | Replace `<YOUR_ANSIBLE_CONTROLLER_URL>` |
| Token URL                 | https://`<YOUR_ANSIBLE_CONTROLLER_URL>`/api/o/token | Replace `<YOUR_SNOW_INSTANCE>`       | Replace `<YOUR_ANSIBLE_CONTROLLER_URL>` |
| OAuth Redirect URL        | https://`<YOUR_SNOW_INSTANCE>`.service-now.com/api/sn_ansible_spoke/ansible_oauth_redirect | Replace `<YOUR_SNOW_INSTANCE>`          |

Then click on the `Create and Get OAuth Token` Button

![Service Now](/images/snow-4.png)

6. A windows will popup , click on `Authorize` 

TODO: Continue Setup

## Creating A Simple catalog Item

TODO: Add Details

## Creating A Simple Flow

TODO: Add Details

## Configuring the Catalog Item to execute the Flow

TODO: Add Details

## Flow Variable Workaround