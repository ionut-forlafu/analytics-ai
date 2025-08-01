# Configure Chat with ATOM LLM

## Introduction

This lab will take you through the steps needed to provision Oracle Digital Assistant & Visual Builder Cloud Service. It will also cover dynamic group and policy creation along with the configuration of the LLM component. 

Document understanding and speech configuration will be covered in the following labs.

Estimated Time: 2 hours 30 minutes

### About Oracle Digital Assistant

Oracle Digital Assistant delivers a complete AI platform to create conversational experiences for business applications through text, chat, and voice interfaces

### Objectives

Provisioning of ODA

In this lab, you will:

* **Provision and Configure ODA Instance**
  * Follow Task 1 to Task 5 to set-up ODA Instance
* **Provision VBCS Instance**
  * Follow Task 6

### Prerequisites

This lab assumes you have:

* An Oracle account
* Access to the Chicago Region
* Administrator permissions or permissions to use the Generative AI family, the AI services family, Digital Assistant, Visual Builder and Identity Domains

## Task 1: Provision Oracle Digital Assistant

This task will help you to create Oracle Digital Assistant under your choosen compartment.

1. Locate Digital Assistant under AI Services

   ![Navigate to Digital Assistant](images/oda_provision_1.png)

   > **Note:** You can find Digital Assistant under the AI Services.

2. Provide the information for **Compartment**, **Name** , **Description** (optional) & **Shape**. Click **Create**

    ![Create ODA](images/oda_provision_3.png)

3. In few minutes the status of recently created Digital Assistant will change from **Provisioning** to **Active**

    ![Active ODA Instance](images/oda_provision_4.png)

## Task 2: Dynamic Group & Policy creation for Oracle Digital Assistant

This task will help you to create desired dynamic group & necessary policy for the Oracle Digital Assistant

Create a Dynamic Group
Go to Identity>>Domains>>Default domain>>Dynamic groups

![Navigate to Domains](images/domain.png)

Click on Create dynamic group and name it as odaDynamicGroup

Select radio button - Match any rules defined below
Add the following rules. Please change the values of OCIDs to your own values here.

Rule 1

```text
     <copy>
    All {instance.id = 'ocid1.odainstance.oc1.us-chicago-1.XXXX'}
     </copy>
```

Rule 2

```text
     <copy>
    All {resource.type='odainstance', resource.compartment.id='ocid1.compartment.oc1..XXXX' }
    </copy>
 ```

Rule 3

```text
    <copy>
    ALL {resource.type = 'fnfunc', resource.compartment.id = 'ocid1.compartment.oc1..XXXX'}
     </copy>
```

1. Attach the policy at the root compartment level. Please change the values of OCIDs to your own values here.

    ```text
    <copy>
    Allow any-user to use ai-service-generative-ai-family in tenancy where request.principal.id='ocid1.odainstance.oc1.us-chicago-1.XXXXXXXXXXXXXXXXXXXXXXXXXX'
    Allow any-user to use generative-ai-family in tenancy where request.principal.id='ocid1.odainstance.oc1.us-chicago-1.XXXXXXXXXXXXXXXXXXXXXX'
    Allow any-user to use fn-invocation in tenancy where request.principal.id='ocid1.odainstance.oc1.us-chicago-1.XXXXXXXXXXXXXXXXXXXX'
    Allow dynamic-group odaDynamicGroup to use fn-invocation in tenancy
    </copy>
    ```

    > **Note:**
    > * Please make sure that the compartmentId should be the one under which the resource is  created.

## Task 3: Create LLM REST Service for the OCI Generative AI Service

This task involves creating REST service which will be used by ODA to connect to OCI Generative AI Service. The REST Service will be created for the ODA created in **Task 1**.  This step only needs to be done once per ODA instance. If users are sharing one ODA instance to create multiple chatbot, only the first person will need to perform this step

1. Locate the ODA created in **Task 1**

    ![ODA List](images/oda_list.png)

2. Select the earlier created ODA Instance and click on **Service Console**

    ![ODA Service Console](images/oda_provision_4.png)

3. Click on hamburger menu and locate & click **API Services**

    ![API Services](images/oda_api_service.png)

4. Click on **Add LLM Service**. Provide the following details. Please note you will have to change values of CompartmentID and modelID to your own ID values in the Body section. You can follow the next step - Step 5 to see how to retrieve model ID.
    * **Name**

    ```text
    <copy>
    Gen_AI_Service
    </copy>
    ```

    * **Endpoint**

    ```text
    <copy>
    https://inference.generativeai.us-chicago-1.oci.oraclecloud.com/20231130/actions/chat
     </copy>
    ```

    * **Description (Optional)** : `Optional`
    * **Authentication Type** : OCI Resource Principal
    * **Method** : POST
    * **Request**
    * **Body**

    ```text
    <copy>
    {
        "compartmentId": "ocid1.compartment.oc1..XXXXXXXXXXX",
        "servingMode": {
            "modelId": "ocid1.generativeaimodel.oc1.us-chicago-1.XXXXXXXX",
            "servingType": "ON_DEMAND"
        },
        "chatRequest": {
            "apiFormat": "COHERE",
            "message": "Hi, how are you",
            "isStream": true
        }
    }
    </copy>
    ```

5. This step is broken down into following 3 steps
    * Step 1: Click on hamburger menu of OCI console and select **AI Services** > **Generative AI**

    ![API Services](images/genai.png)

    * Step 2: Select Chat (under **Playground** heading)

    ![API Services](images/model_screenshot.png)

    * Step 3: For the **Model**=**cohere.command-r-plus v1.7**, Click **View Model Details**, and then click on **copy** link for the **cohere.command-r-plus** and **version** = 1.7

        > **Note:** v1.7 is the latest model as of this livelab release. This lab should work with any version of the cohere model. 

    ![API Services](images/chat_screenshot.png)

6. Click **Test Request** to make sure the connection is successful

   ![API Services](images/oci_rest_service_3.png)

    > **Note**
    > * Retrieve the modelId (OCID) from OCI Gen AI Services Playground and use a compartmentId where the ODA is hosted inside
    > * If you are using a different name (and not Gen AI Service) for your Rest service then please make a change in your LLM Provider in Settings as well. To do that Go to Skills -> Settings -> Configuration -> Large Language Model Services -> LLM Provider. Choose the new Rest Service for the GenAI LLM 

    ![API Services](images/oci_rest_service_4.png)

## Task 4: Import Skill (Provided)

1. Click on the link to download the required skill (zip file): [Atom Skill.zip](https://objectstorage.us-chicago-1.oraclecloud.com/n/idb6enfdcxbl/b/Livelabs/o/docunderstanding%2FATOM_Livelab(1.0.1).zip)

2. Import the skill (downloaded). Click on **Import Skill** & select the zip file to import

   ![Import Skill](images/import_skill.png)

3. Once the skill is imported. Click on the Skill and go to Components.

4. Edit the R Transformer by selecting the pencil icon in the top right 

    ![Open R Transformer](images/edit-r-transformer.png)

5. Edit the model and compartment id to your own 

    ![Edit Id](images/edit-comp-model-id.png)

6. Go to Skills -> Settings -> Configuration -> Large Language Model Services. Configure the LLM Service.

    ![API Services](images/oci_rest_service_4.png)

7. Configure the LLM Provider value as the one you configured in Task 3. Use the R Transformer as the transformation handler. Click on Check mark under Action to save it as shown in the image below.

    ![LLM Service](images/lllm-services-skill-config.png)

8. Go to Skills -> Flows. Click on Chat.

    ![Chat Services](images/chat.png)

9. Click on invokeLLM and then click on Component. Select the same LLM Service which was created in Step 7.

    ![Invoke LLM](images/invoke_llm.png)

## Task 5: Create Channel to embed ODA in Visual Builder Application (provided) or in any custom Web App

1. Click on hamburger menu and select Development > Channels

    ![Create Channel](images/create_channel.png)

2. Select the following option on the form:

    * **Channel Type** = Oracle Web
    * **Allowed Domain** = *

    ![Create Channel](images/create_channel_1.png)

3. After channel creation, enable the Channel by using the toggle button (screenshot).
   * Route it to skill imported in Task 4

   ![Create Channel](images/route_skill1.png)

4. Disable the **Client Authentication Enabled** toggle. (Take note of channelId for **Task 6** in later step).

    ![Create Channel](images/skill_channel1.png)

## Task 6: Create VBCS Instance & embed ODA skill in VBCS Application (Please directly move to Step 5 incase you already have a VBCS instance provisioned)

1. Click on main hamburger menu on OCI cloud console and navigate Developer Services > Visual Builder

    ![Create Channel](images/visual_builder.png)

2. Create Visual Builder Instance by providing the details and click **Create Visual Builder Instance**:
    * **Name** = <name_of_your_choice>
    * **Compartment** = <same_compartment_as_oda>
    * **Node** = <as_per_need>

    ![Create Channel](images/create_vbcs.png)

3. Wait for the instance to come to **Active** (green color) status

4. Click on the link to download the VB application (zip file): [ATOM_Training.zip](https://objectstorage.us-chicago-1.oraclecloud.com/n/idb6enfdcxbl/b/Excel-Chicago/o/Livelabs%2Fdoc-understanding%2FATOM_Training-1.0.1.zip)

5. Import the application in provisioned instance as per the screenshots. Users only need one VCBS instance created. They can import/create multiple applications in the instance for each additional chatbot they have

    * Click on Import from Visual Builder Instance

        ![Create Channel](images/import_vbapp.png)

    * Choose the option as below

        ![Create Channel](images/import_vbapp_1.png)

    * Provide the App Name with other details and select the provided application zip file

        ![Create Channel](images/import_vbapp_2.png)

6. Once import is completed, open the embedded-chat javascript file in the VB Instance and update the details as follows:

    * **URI** = '<https://oda-XXXXXXXXXXXXXXXXXXXXXX.data.digitalassistant.oci.oraclecloud.com/>'
    * **channelId** = 'XXXXXXXXXXXXXXXXXXXXXXXXXXXX'
    * Please change value of initUserHiddenMessage on Line 32 from 'what can you do' to 'Hello'

    ![Create Channel](images/vb_config.png)

    > **Note**
    > * URI is the hostname of ODA instance provisioned in **Task 1**
    > * channelId is created during **Task 5** - **Step 3**

7. The UI of the chatbot such as theme, color and icon can be changed by modifying the parameters under var chatWidgetSetting from embedded-chat javscript file.

8. Click on the Play button shown in the above image on the top right corner to launch ATOM chatbot and start chatting with ATOM.

9. If the preview is working as expected, you can open your visual builder application and begin conversing with ATOM 

    ![Converse with ATOM](images/chat-with-atom.png)

**Troubleshooting** 

1. You may face an issue when you go to publish the live link of the application. It may throw a "forbidden" error. The solution is to remove the "Admin" and "User" role in the JSON tab from all the vb pages - main-start, main-embedded-chat, and the shell page as shown in the image below.
    ![VB Error](images/vb_error.png)

2. If you get 404 errors, it's likely a permission issue. Please review the policies. 

> **Note:** (Optional) If you would like to edit the custom components locally in your IDE, you will need to install the bots node sdk

> * Run this command in your terminal to install -
        ```text
        <copy>
        npm install @oracle/bots-node-sdk
        </copy>
        ```
> * Once installed - cd into the folder and run the below command to zip the folder.
        ```text
        <copy>
        npx @oracle/bots-node-sdk pack
        </copy>
        ```

## Acknowledgements

**Authors**

* **Nitin Jain**, Master Principal Cloud Architect, NACIE
* **Abhinav Jain**, Senior Cloud Engineer, NACIE
* **JB Anderson**,  Senior Cloud Engineer, NACIE

**Contributors**
* **Luke Farley**, Senior Cloud Engineer, NACIE

**Last Updated By/Date:**
* **Luke Farley**, Senior Cloud Engineer, NACIE, Apr 2025
