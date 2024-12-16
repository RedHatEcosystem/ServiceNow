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

4. Enter the following information for the new Application form: 

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

1. Log on to Automation Controller
2. On the left panel under `Administration`, click on `Credential Types`

![Automation Controller](/images/controller-5.png)

3. In the main panel, click on the `Add` Button

![Automation Controller](/images/controller-6.png)

4. Enter the following information for the new Credential Type form:

| Key                      | Value                                                                                      | Note                               |
|--------------------------|--------------------------------------------------------------------------------------------|------------------------------------|
| Name                     | ServiceNow                                                                                 |                                    |
| Description              | Credential type for ServiceNow authentication                                              |                                    |

Make sure that `yaml` is selected for both the `Input Configuration` and `Injector Configuration` sections and enter the following:


`Input Configuration`:
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
`Injector Configuration`:

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

Then click `Save`

![Automation Controller](/images/controller-7.png)


5. On the left panel under `Resources`, click on `Credentials`

![Automation Controller](/images/controller-8.png)

3. In the main panel, click on the `Add` Button

![Automation Controller](/images/controller-9.png)

4. Enter the following information for the new Credential form:

| Key                      | Value                                                  | Note                                   |
|--------------------------|--------------------------------------------------------|----------------------------------------|
| Name                     | ServiceNow Instance <YOUR_INSTANCE_ID>                 |                                        |
| Description              | Credential for ServiceNow Instance <YOUR_INSTANCE_ID>  |                                        |
| Service Now Username     | ServiceNow Username                                    |                                        |
| Service Now Password     | ServiceNow Password                                    |                                        |
| Service Now Instance     | <YOUR_INSTANCE_ID>                                     | Just the instance ID, not the full URL |

Then click `Save`

![Automation Controller](/images/controller-10.png)

Now you have a crednetial that you can use to automate ServiceNow tasks using the `servicenow.itsm` and `servicenow.servicenow` collections



# Configuring Your ServiceNow Instance

## Installing the required store applications for Automating ServiceNow and the notification service for EDA

1. Log on to ServiceNow using your `admin` account
2. On the top panel, click on `All` 

![Service Now](/images/snow-40.png)

3. Type in `Company` in the `Filter` field and then choose `My Company Applications` under `System Applications`.

![Service Now](/images/snow-41.png)

4. Click on `Install` next to the application you want to install. 

![Service Now](/images/snow-42.png)

5. The application(s) should then move from the `Not Installed` section and appear under the `Installed` section.

![Service Now](/images/snow-43.png)

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

![Service Now](/images/snow-5.png)


Ansible Spoke is now configured 


## Creating A Simple Flow

1. Log on to ServiceNow using your `admin` account
2. On the left panel type in `Flow` in the `Filter navigator` and Click on `Flow Designer` Under `Process Automation`. The `Flow Designer` will open in a new tab.

![Service Now](/images/snow-1.png)

3. In the `Flow Designer`, click on the `New` button, then choose `Flow`

![Service Now](/images/snow-6.png)

4. Enter the following information in the new flow dialog box:

| Key                      | Value                            | Note                                   |
|--------------------------|----------------------------------|----------------------------------------|
| Flow n                   | Ansible: Print Greeting Message  |                                        |
| Application              | Ansible Spoke                    |                                        |
| Protection               | None                             |                                        |
| Run As                   | System User                      |                                        |

Then Click on `Submit`

![Service Now](/images/snow-7.png)

5. In the new flow page, click on `Add a Trigger`

![Service Now](/images/snow-8.png)

6. Choose `Service Catalog` Under `Application` and click on `Done`

![Service Now](/images/snow-9.png)

7. In the new flow page, click on `Add an Action, Flow Logic, or Subflow`

![Service Now](/images/snow-10.png)

8. Choose `Action`

![Service Now](/images/snow-11.png)

9. Select `Ansible` under `INSTALLED SPOKES` and then choose `Launch Job Template` under `Template Management`

![Service Now](/images/snow-12.png)

10. Enter the Automation Controller Job Template ID. We are planning on creating a Catalog item later that will capture 2 values and store them in 2 variabl andes named `snow_name_input` and `snow_message_input`. To pass the values of these variables to Automation Controller, click on the script button for the `Extra Arguments` field and input the following:

```
var extra = "{ \"extra_vars\":{"
    + "  \"greeting_name_input\": \"" + fd_data.trigger.request_item.variables.snow_name_input.getDisplayValue() + "\","
    + "  \"greeting_message_input\": \"" + fd_data.trigger.request_item.variables.snow_message_input.getDisplayValue() + "\""
    + "}}";
return extra;

```

This will pass 2 extra vars to Automation Controller named `greeting_name_input` and `greeting_message_input` with the values captured from service now.

Click on Done

![Service Now](/images/snow-13.png)

11. Click on `Save`

![Service Now](/images/snow-14.png)

12. Click on `Activate`

![Service Now](/images/snow-15.png)


We now have a flow that is triggered through a Service Catalog item, that will launch a specific job template in Automation Controller.

## Creating A Simple catalog Item

### Create a Catalog Item Template

1. Log on to ServiceNow using your `admin` account
2. On the left panel type in `Catalog Builder` in the `Filter navigator` and Click on `Catalog Builder` Under `Service Catalog`. The `Catalog Builder` will open in a new tab.

![Service Now](/images/snow-16.png)

3. Click on `Create a catalog item template`

![Service Now](/images/snow-17.png)

4. Choose `Standard` and Click on `Continue`

![Service Now](/images/snow-18.png)

5. In the `Template Details` Section, enter the following Info:

| Key                      | Value                            | Note                                   |
|--------------------------|----------------------------------|----------------------------------------|
| Template name            | Ansible Request Template         |                                        |
| Short description        | SNOW to AAP Integration          |                                        |
| Template Available for   | Any User                         |                                        |

![Service Now](/images/snow-19.png)

6. In the `Overrides` Section, Click on Browse:

![Service Now](/images/snow-20.png)

7. Highlight and move all items from the `Available` section to the Selected section and then click on `Save Selection`:

![Service Now](/images/snow-21.png)

7. In the `Review and Submit` section click on `Submit`:

![Service Now](/images/snow-22.png)


### Create a Catalog Item Template

1. Back in the `Catalog Builder`, Click on `create a new catalog item`

![Service Now](/images/snow-23.png)

2. In the `Getting Started` page, click on `Continue`

![Service Now](/images/snow-24.png)

3. Select the template we just created earlier, Choose `Ansible Request Template` and click on `use this item template`

![Service Now](/images/snow-25.png)

4. In the `Details` tab, enter the following information:

| Key                      | Value                            | Note                                   |
|--------------------------|----------------------------------|----------------------------------------|
| Template name            | Ansible: Print Greeting Message  |                                        |
| Short description        | Testing SNOW to AAP Integration  |                                        |
| Image                    | <Upload_an_image>                | Optional                               |
| Description              | <Description_text>               | Optional                               |

![Service Now](/images/snow-26.png)

5. In the `Location` tab, Choose `Service Catalog` under `Catalogs` and `Services` under `Categories`:

![Service Now](/images/snow-27.png)

6. In the `Questions` tab, click on `Insert new question`

![Service Now](/images/snow-28.png)

7. Enter the follwing information in the new question details page:

| Key                      | Value                | Note                                   |
|--------------------------|----------------------|----------------------------------------|
| Question type            | Text                 |                                        |
| Question subtype         | Single-line          |                                        |
| Question label           | Enter your name     |                                        |
| Name                     | `snow_name_input`    | Variable name we used in the flow      |
| Mandatory                | Checked              |                                        |

click on `Insert Question`

![Service Now](/images/snow-29.png)

8. In the `Questions` tab, Repeat the process again to add another question. Enter the follwing information in the new question details page:

| Key                      | Value                | Note                                   |
|--------------------------|----------------------|----------------------------------------|
| Question type            | Text                 |                                        |
| Question subtype         | Single-line          |                                        |
| Question label           | Enter your message   |                                        |
| Name                     | `snow_message_input` | Variable name we used in the flow      |
| Mandatory                | Checked              |                                        |

Your `Questions` tab should now look like this

![Service Now](/images/snow-30.png)

9. In the `Settings` tab, choose `Submit` for the button label, and check all available options

![Service Now](/images/snow-31.png)

10. In the `Access` tab, Make sure that `Any User` is selected under `Available for`

![Service Now](/images/snow-32.png)

11. In the `Fulfillment` tab, Choose `Flow Designer flow` under `Process engine`, and choose the flow we created earlier, `Ansible: Print Greeting Message` under `Selected flow`

![Service Now](/images/snow-33.png)

12.  In the `Review and Submit` tab, click on `Submit`

![Service Now](/images/snow-34.png)

We have finished creating a catalog item in the service catalogue

### Testing the Catalog Item

1. On the left panel type in `Service Catalog` in the `Filter navigator` and Click on `Service Catalog` Under `Self-Service`, Then click on `Services` Under  `Service Catalog`.

![Service Now](/images/snow-35.png)

2. You should find an entry for the Catalog item we created named `Ansible: Print Greeting Message`, click on it

![Service Now](/images/snow-36.png)

3. Enter values for the request (`name` and `message`) and click on `Order Now`

![Service Now](/images/snow-37.png)

4. Wait for a confirmation that the request has been submitted, the flow is now running and should trigger a new Job on Automation Controller

![Service Now](/images/snow-38.png)

5. In Controller, you should see the job, and should be able to see the values from the form in the output 

![Service Now](/images/snow-39.png)

The playbook used for this workflow is:

```yaml
---
- name: Print a greeting message
  hosts: all
  gather_facts: false
  vars:
    greeting_name: "{{ greeting_name_input | default('Ansible User') }}"
    greeting_message: "{{ greeting_message_input | default('Have a great day!') }}"
  tasks:
  - name: Print Greeting Message
    ansible.builtin.debug:
      msg: "Hello {{ greeting_name }}, {{ greeting_message }}"
```