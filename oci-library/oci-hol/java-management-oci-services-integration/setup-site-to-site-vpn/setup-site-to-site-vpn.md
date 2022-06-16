# Set up Site-to-Site VPN

## Introduction

Site-to-Site VPN provides a site-to-site IPSec connection between your on-premises network and your virtual cloud network (VCN). The IPSec protocol suite encrypts IP traffic before the packets are transferred from the source to the destination and decrypts the traffic when it arrives. Site-to-Site VPN was previously referred to as VPN Connect and IPSec VPN.

In this lab, you will follow the scenario where an on-premises network would have private access to the Oracle Services such as JMS using Site-to-Site VPN.

The set up image is given for your reference.
![image of on-premises network private access to oracle services](/../images/network-sgw-transit-basic-layout-with-gateways.png)

This lab walks you through the steps to set up Site-to-Site VPN between your on-premises network and your virtual cloud network (VCN). 

Estimated Time: 1 hour

### Objectives

In this lab, you will:

* Set up VCN and additional networking components 
* Set up CPE object
* Set up IPSec connection
* Configure on-premises CPE


### Prerequisites

* You have signed up for a paid account with Oracle Cloud Infrastructure and have received your sign-in credentials.
* Access to the cloud environment and resources configured in the [Manage Java Runtimes, Applications and Managed Instances Inventory with Java Management Service Workshop](https://apexapps.oracle.com/pls/apex/dbpm/r/livelabs/view-workshop?wid=912).

 > **Note:** This lab requires paid OCI account as Service Gateway is not available in Free tier.


## Task 1: Gather information

Before getting started with set up of Site-to-Site VPN, you must gather the following information.
* What is your VCN's CIDR?

    * A VCN covers one or more IPv4 CIDR blocks or IPv6 prefixes of your choice. The allowable VCN size range is /16 to /30. Example: 10.0.0.0/16. For this set up we will keep VCN CIDR **20.0.0.0/16**.

* What is the public IP address of your CPE device?
    * CPE or Customer-Premises Equipment is the VPN service on your on-premises side. CPE will be configured later in this lab, for now you can run the following command on your on-premises host where you intend to configure CPE.

     ```
    <copy>
    ifconfig
    </copy>
    ```  
    Take note of the Public IP address.

* What is the CIDR range of the on-premises network?
    * You can use the following command to check the private IP address of the host.
    ```
    <copy>
    ifconfig
    </copy>
    ``` 

        Once you have obtained the private IP address, add /32 behind it to make a CIDR block. For example 192.168.11.22**/32**.

## Task 2: Set up VCN and additional networking components

1. Sign in to the Oracle Cloud Console as an administrator using the credentials provided by Oracle, as described in [Signing into the Console](https://docs.oracle.com/en-us/iaas/Content/GSG/Tasks/signingin.htm).
&nbsp;

2. Create the VCN
    * Open navigation menu, click **Networking**, and then click **Virtual Cloud Networks** option. 
    ![image of navigating to Virtual Cloud Networks section](/../images/navigate-to-vcn.png)

    * Select the compartment where you want to create the VCN. In this case, select `Fleet_Compartment`.

    * Click **Create VCN**.
    ![image of step to create a new VCN](/../images/create-vcn.png)

    * Enter the following values:
        * **Create in Compartment**: Leave as is.
        * **Name**: A descriptive name for the cloud network. 
        * **IPv4 CIDR Block**: 20.0.0.0/16 as mentioned in Task 1.

        ![image of details for new VCN](/../images/details-for-new-vcn.png)

    * Click **Create VCN**.
     ![image of saving a new VCN](/../images/save-new-vcn.png)

 
3. Create the Dynamic Routing Gateway (DRG)
    * Open the navigation menu and click **Networking**. Under **Customer Connectivity**, click **Dynamic Routing Gateway**.
    ![image of navigating to Dynamic Routing Gateway section](/../images/navigate-to-dynamic-routing-gateway.png)

    * Click **Create Dynamic Routing Gateway**.
    ![image of step to create a new Dynamic Routing Gateway](/../images/create-new-drg.png)

    * Enter the following values:
        * **Create in Compartment**: Leave as is (the VCN's compartment).
        * **Name**: A descriptive name for the DRG.

        ![image of details for new VDRGCN](/../images/details-for-new-drg.png) 
        
    * Click **Create Dynamic Routing Gateway**.
    ![image of saving a new DRG](/../images/save-new-drg.png)

4. Attach the DRG to the VCN
    * Click the name of the DRG you created in previous step.
    ![image of newly created DRG](/../images/newly-created-drg.png)

    * Under **Resources**, click **Virtual Cloud Networks Attachments**.

    * Click **Create Virtual Cloud Network Attachment**.
    ![image of creating new VCN attachment](/../images/create-vcn-attachment.png)

    * Select the VCN that you created in step 1. 
    ![image of details of VCN attachment](/../images/details-for-new-drg-att.png)


    * Click **Create Virtual Cloud Network Attachment**.
    ![image of save new VCN attachment](/../images/save-new-drg-att.png)


5. Create the Service Gateway (SWG)
    * Open navigation menu, click **Networking**, and then click **Virtual Cloud Networks** option.
    ![image of navigating to Virtual Cloud Networks section](/../images/navigate-to-vcn.png)

    * Select the VCN you have created in step 1.
    * Under **Resources**, click **Service Gateway**.
    * Click **Create Service Gateway**.
    ![image of save new Service Gateway](/../images/create-new-service-gateway.png)


    * Enter the following values:
        * **Create in Compartment**: Leave as is (the VCN's compartment).
        * **Name**: A descriptive name for the SWG. 
        * **Services**: Select **All IAD Services In Oracle Services Network** from the drop down list.

        ![image of details for new Service Gateway](/../images/details-for-service-gateway.png)
      * Click **Create Service Gateway**.
      ![image of save new Service Gateway](/../images/save-service-gateway.png)

6. Create a Route Table and Route Rule for SWG

    * Open the navigation menu, click **Networking**, and then click **Virtual Cloud Networks**.
    ![image of navigating to Virtual Cloud Networks section](/../images/navigate-to-vcn.png)

    * Select the VCN you have created in step 1.
    * Under **Resources**, click **Route Tables**.
    * Click **Create Route Table**.
    ![image of creating new Route Table](/../images/create-new-route-table.png)
    
    * Enter the following values:
        * Name: A descriptive name for the route table 
        * Create in compartment: Leave as is.
        * Click **+ Another Route Rule**, and enter the following:
            * **Target Type**: Service Gateway.
            * **Destination Service**: Select **All IAD Services In Oracle Services Network** from the drop down list.
            * **Compartment**: The compartment where the service gateway is located.
            * **Target**: The service gateway that you have created.
            * **Description**: An optional description of the rule.

        ![image of creating new Route Table](/../images/route-table-for-swg.png)
                
    * Click Create Route Table.
    ![image of saving new Route Table](/../images/save-changes-route-table-for-swg.png)

7. Create a Route Table and Route Rule for DRG

    * On the same **Route Tables** page, click **Create Route Table**.
     ![image of creating new Route Table](/../images/create-new-route-table.png)

    * Enter the following values:
        * **Name**: A descriptive name for the route table 
        * **Create in compartment**: Leave as is.
        * Click **+ Another Route Rule**, and enter the following:
            * **Target Type**: Dynamic Routing Gateway. 
            * **Destination CIDR Block**: The CIDR for your on-premises network (see the list of information gathered in Task 1).
            * **Description**: An optional description of the rule.
        
        ![image of details of new Route Table](/../images/route-table-for-drg.png)
        
    * Click **Create**.
    ![image of saving new Route Table](/../images/save-changes-route-table-for-drg.png)

    * Both the newly created Route Tables should look like this.
    ![image of both new Route Tables](/../images/all-route-tables.png)



8. Add SWG Route Rule in DRG attachment
    * Open the navigation menu, click **Networking**, and then click **Virtual Cloud Networks**.
    ![image of navigating to Virtual Cloud Networks section](/../images/navigate-to-vcn.png)

    * Select the VCN you have created in step 1.
    * Under **Resources**, click **Dynamic Routing Gateways Attachments**.
    * Click the Attachment that you have created.
    ![image of dynamic routing gateway page](/../images/dynamic-routing-gateways-attachments-page.png)

    * Click **Edit**.
     ![image of editing DRG](/../images/edit-drg-attachment.png)

    * Click **Show Advanced Options**.
    ![image of configuring  DRG](/../images/show-advanced-options-drg-att.png)

    * Toggle to **VCN route table** tab.
    ![image of configuring DRG](/../images/toggle-to-vcn-route-table.png)

    * Select **Select Existing** radio button and then select the Route rule for DRG from drop down list, that you have created.
    ![image of configuring DRG](/../images/select-route-rule.png)

    * Click **Save Changes**.
    ![image of saving the configuring of DRG](/../images/save-new-route-rule.png)


9. Add DRG Route Rule in SWG
    * Open the navigation menu, click **Networking**, and then click **Virtual Cloud Networks**.
    ![image of navigating to Virtual Cloud Networks section](/../images/navigate-to-vcn.png)

    * Select the VCN you have created in step 1.
    * Under **Resources**, click **Service Gateway**.
    ![image of service gateway page](/../images/select-service-gateway.png)

    * Click on more options button and select **Associate Different Route Table**.
    ![image of editing service gateway](/../images/edit-service-gateway.png)


    
    * Select the Route rule for DRG from drop down list, that you have created.
    ![image of configuring SWG](/../images/associate-route-rule-to-swg.png)

    * Click **Associate Different Route Table**.
    ![image of saving configuration SWG](/../images/save-associate-route-rule-to-swg.png)


## Task 3: Setup a CPE object and IPSec connection

In this task, you will create the CPE object and IPSec tunnels and configure the type of routing for them (static routing). CPE object is a virtual representation (on Oracle side) of your CPE device (on on-premises network).

1. Create CPE object

    * Open the navigation menu and click **Networking**. Under **Customer Connectivity**, click **Customer-Premises Equipment**.
    ![image of navigating to Customer connectivity section](/../images/navigate-to-customer-connectivity.png)

    * Click **Create Customer-Premises Equipment**.
     ![image of creating new Customer Premises Equipment](/../images/create-cpe.png)

    * Enter the following values:
        * **Create in Compartment**: Leave as is (the VCN's compartment).
        * **Name**: A descriptive name for the CPE object.
        * **IP Address**: The public IP address of the actual CPE/edge device at your end of the VPN (see the list of information to gather in Task 1).
        * **Vendor**: Select Libreswan.
        * **Platform/Version**: Select the latest version available.

        ![image of details of new Customer Premises Equipment](/../images/details-of-new-cpe.png)

    * Click **Create CPE**.
    ![image of saving new Customer Premises Equipment](/../images/save-details-of-cpe.png)


2. Create IPSec Connection
    * Open the navigation menu and click **Networking**. Under **Customer Connectivity**, click **Site-to-Site VPN**.
    ![image of navigating to Customer connectivity section](/../images/navigate-to-site-to-site.png)

    * Click **Create IPSec Connection**.
    ![image of creating new IPSec Connection](/../images/create-ipsec-connection.png)


    * Enter the following values:

        * **Create in Compartment**: Leave as is (the VCN's compartment).
        * **Name**: Enter a descriptive name for the IPSec connection. 
        * **Customer-Premises Equipment Compartment**: Leave as is (the VCN's compartment).
        * **Customer-Premises Equipment**: Select the CPE object that you created earlier.
        * **Dynamic Routing Gateway Compartment**: Leave as is (the VCN's compartment).
        * **Dynamic Routing Gateway**: Select the DRG that you created in Task 2.
        * **Static Route CIDR**: The CIDR for your on-premises network (see the list of information gathered in Task 1).

        ![image of details of IPSec Connection](/../images/details-of-ipsec.png)




    * On the Tunnel 1 tab:

        * **Tunnel Name**: Enter a descriptive name for the tunnel (optional).
        * **Routing Type**: Select Static Routing.

        ![image of details of IPSec Connection tunnel 1](/../images/details-of-tunnel-1.png)


    * On the Tunnel 2 tab:

        * **Tunnel Name**: Enter a descriptive name for the tunnel (optional).
        * **Routing Type**: Select Static Routing.

        ![image of details of IPSec Connection tunnel 2](/../images/details-of-tunnel-2.png)



    * Click **Create IPSec Connection**.
    ![image of saving IPSec Connection](/../images/save-ipsec-details.png)


    * Created IPSec connection displays tunnel information that includes:

        * The Oracle VPN IP address (for the Oracle VPN headend).
        * The tunnel's IPSec status.
        * To view the tunnel's shared secret, click the tunnel to view its details, and then click Show next to Shared Secret.

    
    * Copy the **Oracle VPN IP address** and **shared secret** for each of the tunnels so you can configure the CPE device on on-premises host in next task.

        * Oracle VPN IP address
        ![image of Oracle VPN IP address](/../images/oracle-vpn-ip-address-details.png)

        * Shared secret
        ![image of Shared secret](/../images/secret-key-details.png)



## Task 4: Configure CPE on your on-premises host
On your on-premises network you need to install Libreswan as a CPE ( Customer-Premises Equipment) so traffic can flow between your on-premises network and virtual cloud network (VCN) via Site-to-Site VPN.

1. Install Libreswan on your on-premises host.
    ```
    <copy>
    sudo yum -y install libreswan
    </copy>
    ```   

2. Depending on the Linux distribution you're using, you might need to enable IP forwarding on your interface to allow clients to send and receive traffic through Libreswan.

3. In the Terminal window, edit the `sysctl.conf` by entering this command:
    ```
    <copy>
    sudo nano /etc/sysctl.conf
    </copy>
    ```  
    In the file, paste the following text:

    ```
    <copy>
    net.ipv4.ip_forward=1
    net.ipv4.conf.all.accept_redirects = 0
    net.ipv4.conf.all.send_redirects = 0
    net.ipv4.conf.default.send_redirects = 0
    net.ipv4.conf.eth0.send_redirects = 0
    net.ipv4.conf.default.accept_redirects = 0
    net.ipv4.conf.eth0.accept_redirects = 0
    </copy>
    ```  
    If you're using an interface other than **eth0**, change **eth0** in the following example to your interface (lines 5 and 7).

     To save the file, type CTRL+x. Before exiting, nano will ask you if you wish to save the file: Type y to save and exit, type n to abandon your changes and exit.

4. Apply the updates with:
    ```
    <copy>
    sudo sysctl -p
    </copy>
    ``` 
5. The Libreswan configuration uses the following variables. Determine the values before proceeding with the configuration.

    * **${cpeLocalIP}**: The private IP address of your Libreswan device (see list of gathered information in Task 1).
    * **${cpePublicIpAddress}**: The public IP address for Libreswan (see list of gathered information in Task 1). 
    * **${oracleHeadend1}**: For the first tunnel, the Oracle public IP endpoint obtained from the Oracle Console (refer to Task 3).
    * **${oracleHeadend2}**: For the second tunnel, the Oracle public IP endpoint obtained from the Oracle Console (refer to Task 3).
    * **${vti1}**: The name of the first VTI used. For example, vti1.
    * **${vti2}**: The name of the second VTI used. For example, vti2.
    * **${sharedSecret1}**: The pre-shared key for the first tunnel (refer to Task 3). 
    * **${sharedSecret2}**: The pre-shared key for the second tunnel (refer to Task 3). 

7. In the Terminal window, edit the `/etc/ipsec.d/oci-ipsec.conf` by entering this command:
    ```
    <copy>
    sudo nano /etc/ipsec.d/oci-ipsec.conf
    </copy>
    ```  
    In the file, paste the following text and replace variable values:
    ```
    <copy>
    conn oracle-tunnel-1
     left=${cpeLocalIP}
     leftid=${cpePublicIpAddress} # See preceding note about 1-1 NAT device
     right=${oracleHeadend1}
     authby=secret
     leftsubnet=0.0.0.0/0 
     rightsubnet=0.0.0.0/0
     auto=start
     mark=5/0xffffffff # Needs to be unique across all tunnels
     vti-interface=${vti1}
     vti-routing=no
     ikev2=no # To use IKEv2, change to ikev2=insist 
     ike=aes_cbc256-sha2_384;modp1536
     phase2alg=aes_gcm256;modp1536
     encapsulation=yes
     ikelifetime=28800s
     salifetime=3600s
conn oracle-tunnel-2
     left=${cpeLocalIP}
     leftid=${cpePublicIpAddress} # See preceding note about 1-1 NAT device
     right=${oracleHeadend2}
     authby=secret
     leftsubnet=0.0.0.0/0
     rightsubnet=0.0.0.0/0
     auto=start
     mark=6/0xffffffff # Needs to be unique across all tunnels
     vti-interface=${vti2}
     vti-routing=no
     ikev2=no # To use IKEv2, change to ikev2=insist 
     ike=aes_cbc256-sha2_384;modp1536
     phase2alg=aes_gcm256;modp1536 
     encapsulation=yes
     ikelifetime=28800s
     salifetime=3600s
    </copy>
    ```  

    To save the file, type CTRL+x. Before exiting, nano will ask you if you wish to save the file: Type y to save and exit, type n to abandon your changes and exit.


7. Now, in the Terminal window, edit the `/etc/ipsec.d/oci-ipsec.secrets` by entereing this command:
    ```
    <copy>
    sudo nano /etc/ipsec.d/oci-ipsec.secrets
    </copy>
    ```  
    In the file, paste the following text and replace variable values:
    ```
    <copy>
    ${cpePublicIpAddress} ${oracleHeadend1}: PSK "${sharedSecret1}"
    ${cpePublicIpAddress} ${oracleHeadend2}: PSK "${sharedSecret2}"
    </copy>
    ```  

    To save the file, type CTRL+x. Before exiting, nano will ask you if you wish to save the file: Type y to save and exit, type n to abandon your changes and exit.


8. Restart Libreswan service.

    ```
    <copy>
    sudo service ipsec restart
    </copy>
    ```  
9. Use the following `ip` command to create static routes that send traffic to your OSN through the IPSec tunnels.The full IP address ranges are defined [here](https://docs.oracle.com/en-us/iaas/tools/public_ip_ranges.json).

    ```
    <copy>
    sudo ip route add ${OSNCidrBlock} nexthop dev ${vti1} nexthop dev ${vti2}
    ip route show
    </copy>
    ```  

     > **Note:** Manual addition to IP route table does not persist on restart. 

## Task 5: Setup Managemnet Gateway and Management Agent
Once you have finishing setting up Site-to-Site VPN, you can install and configure Management Gateway on same host as Libreswan and Management Agent on other on-premises host (in the same network) to demonstrate the working of Site-to-Site VPN setup along with JMS.

* Follow [Manage Java Runtimes, Applications and Managed Instances Inventory with Java Management Service Workshop, Lab 8](https://apexapps.oracle.com/pls/apex/dbpm/r/livelabs/workshop-attendee-2?p210_workshop_id=912&p210_type=3&session=109157515005098) to install, configure and verify Management Gateway and Management Agent.


## Task 6: Verify the Site-to-Site VPN set up

1. Verify IPSec connection
    * Open the navigation menu and click **Networking**. Under **Customer Connectivity**, click **Site-to-Site VPN**.
    ![image of navigating to Customer connectivity section](/../images/navigate-to-site-to-site.png)

    * Select the IPSec Connection that you have created.

    * Check **IPSec Status**
    ![image of verification of IPSec connection](/../images/verify-ipsec-connection.png)

        The **IPSec Status** for both tunnels should be **Up**.

2. Verify Management Gateway and Agent installation
    * In the Oracle Cloud Console, open the navigation menu, click **Observability & Management**, and then click **Agents** under **Management Agent**.
    ![image of console navigation to access management agent overview](/../images/management-agent-overview.png)

    * From the Agents and Gateway list, look for the gateway and agent that was recently installed.The **Availability** should be **Active**.
    ![image of agents main page](/../images/agent-and-gateway-page.png)

3. Testing the setup by blocking traffic at Service Gateway
    * Open navigation menu, click **Networking**, and then click **Virtual Cloud Networks** option.
    ![image of navigating to Virtual Cloud Networks section](/../images/navigate-to-vcn.png)

    * Select the VCN you have created.
    * Under **Resources**, click **Service Gateway**.
    * Select the Service Gateway that you created and click more options (three dots).
    ![image of more options for SWG](/../images/service-gateway-block-traffic.png)

    * Select `Block traffic`.
    * Service Gateway **State** will change from **Available** to **Blocked**.
    ![image of changed state](/../images/blocked-service-gateway.png)

    * After 5-10 mins, on the Oracle Cloud Console, open the navigation menu, click **Observability & Management**, and then click **Agents** under **Management Agent**.
    ![image of console navigation to access management agent overview](/../images/management-agent-overview.png)

    * From the Agents and Gateway list, look for the gateway and agent that was recently installed. The **Availability** should be **Silent**.
    ![image of agents main page](/../images/silent-agent-and-gateway.png)





You may now **proceed to the next lab.**


## Learn More

* If the problem still persists or if the problem you are facing is not listed, please refer to the [Getting Help and Contacting Support](https://docs.oracle.com/en-us/iaas/Content/GSG/Tasks/contactingsupport.htm) section or you may open a a support service request using the **Help** menu in the OCI console.

## Acknowledgements

* **Author** - Bhuvesh Kumar, Java Management Service
* **Last Updated By** - Bhuvesh Kumar, June 2022
