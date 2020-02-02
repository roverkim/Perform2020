![perform](./images/perform_logo.png)

# Auto-Remediation with Ansible Tower

## Prerequisites

TODO: add instructions with images how to get the API and PaaS token from dynatrace

## Install tools and demo application

1. Edit the creds.json file and provide your own credentials.

    ```console
    nano creds.json
    ```
    You can save by ctrl+x and exit with ctrl+x

1. Deploy the Dynatrace OneAgent Operator with this script:

    ```console
    ./1-deployDynatraceOperator.sh
    ```

1. Check if everything is up and running:
    ```
    kubectl get pods -n dynatrace 
    ```

1. Deploy the demo application for our workshop with the provided script:
    ```console
    ./2-setupSockShop.sh 
    ```

1. Fetch the public IP address for the carts service and copy to a temporary file - we will need it later.
    ```console
    kubectl get services -n production
    ```

1. Setup the Ansible Tower with a provided script. This will setup Ansible Tower and configure some job templates that we will need for this workshop. 
    ```console
    ./3-installAnsible.sh
    ```
    Please save the following Ansible Job URL to a temporary file.

    In case you are running in any issues during this step, follow the troubleshooting guide at the end of this tutorial.

1. Open the Ansible Tower by copying the **Tower URL** to a new browser tab. The login credentials are `admin / dynatrace`. 

# Configure Dynatrace
    
1. Create tags in Dynatrace via **Settings -> Tags -> Automatically applied tags**

    1. Create a tag named "**app**"
        - Optional tag value: `{ProcessGroup:KubernetesContainerName}` 
        - Rule applies to: `process groups` 
        - Conditions: `Kubernetes container name` `exists`
        
        ![tag app](./images/tag-app.png)


    1. Create a tag named "**environment**" 

        - Optional tag value: `{ProcessGroup:KubernetesNamespace}` 
        - Rule applies to: `process groups`
        - Conditions: `Kubernetes container name` `exists`

        ![tag env](./images/tag-environment.png)
    

1. Create a Problem notification in Dynatrace via **Settings->Integrations-> Problem notifications -> Ansible**
    - Name: "Ansible Tower"
    - Template Url: insert the url of Step 4 here
    - Credentials: `admin / dynatrace`

    ![problem notification](./images/problem-notification.png)


# Let's try to mess with the service

1. Start the load-generation script with this command. Please exchange the IP for the carts service you saved earlier in this tutorial.

    ```console
    ./add-to-carts.sh IP-OF-CARTS-SERVICE
    ```

1. Login to your Ansible Tower to start the prepared demo workflow. 

1. Navigate to the **Resources -> Templates** section and start the provided template named **start-campain**

    ![start-campaign](./images/start-campaign.png)

1. Change the **promotion rate** to **30** and **LAUNCH** the job template.

    ![launch](./images/launch.png)

1. This will trigger a promotional campaign where 30 % of all user interactions of a shopping card _should_ receive a promotional item. Instead, what happens is that we will see an increase in the failure rate since the campaign has not been implemented yet. 

1. Watch the self-healing take place ;)



# Troubleshooting

- In case Ansible Tower did not successfully install, please execute the following script to delete the failed installation and to try again:
    ```
    kubectl delete ns tower
    ```
    ```
    ./3-installAnsible.sh
    ```