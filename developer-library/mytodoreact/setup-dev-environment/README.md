## MyToDoReact version 1.0.
Copyright (c) 2021 Oracle, Inc.

Licensed under the Universal Permissive License v 1.0 as shown at https://oss.oracle.com/licenses/upl/

# Part 1 --  Setup OCI, Cloud shell, OKE, ATP and

## **Summary**

In this part of the lab, you will configure your development environment and collect information that will be used later throughout this workshop

### Objectives of Part I

* Setup an OCI account
* Launch Cloud Shell
* Download the workshop code and scripts from GitHub
* Set up an OCI Compartment and install a two nodes OKE cluster
* Create the ATP database, the user schema and a database table
* Install GraalVM  
* Create an OCI Registry and Auth key

## **STEP 1**: Setup your OCI Account and Launch the Cloud Shell

 1. An Oracle Cloud paid account or free trial with credits. To sign up for a trial account with $300 in credits for 30 days, click [here](http://oracle.com/cloud/free).
 You will not be able to complete this workshop with the 'Always Free' account. Make sure that you select the free trial account with credits.

2. Launch Cloud Shell

Cloud Shell is a small virtual machine running a Bash shell which you access through the OCI Console. It comes with a pre-authenticate CLI pre-installed and configured so you can immediately start working in your tenancy without having to spend time on installation and configuration!

Click the Cloud Shell icon in the top-right corner of the Console.

  ![](images/7-open-cloud-shell.png " ")


## **STEP 2**: Clone mtdrworkshop GitHup repository

1. Open up Cloud Shell and clone the GitHub repo.

    ````
    <copy>
    git clone https://orahub.oci.oraclecorp.com/ora-jdbc-dev/mtdrworkshop.git
    </copy>
    ````

  You should now see `mtdrworkshop` in your root directory

2. Change to the mtdrworkshop directory:

    ```
    <copy>cd mtdrworkshop</copy>

    ```
3. Set the execution mode for all Shell scripts

```
<copy>chmod +x *.sh */*.sh</copy>
```
## **STEP 3**: Create an OCI compartment and an OKE cluster in that compartment

 1. Open up the hamburger menu in the top-left corner of the Console and select **Identity > Compartments**.

  ![](images/15-identity-compartments.png " ")

 2. Click **Create Compartment** with the following parameters then click **Create Compartment**:
        - Compartment name: `mtdrworkshop`
        - Description: `My ToDo React workshop compartment`

  ![](images/16-create-compartment.png " ")

  ![](images/17-create-compartment2.png " ")

  3. Once the compartment is created, click the name of the compartment and then click **Copy** to copy the OCID.

  ![](images/19-compartment-name-ocid.png " ")

  ![](images/20-compartment-ocid.png " ")

  4. Go back into your cloud shell and verify you are in the `~/mtdrworkshop` directory.

  5. Run `./setCompartmentId.sh <COMPARTMENT_OCID> <REGION_ID>` where your `<COMPARTMENT_OCID>` and `<REGION_ID>` values are set as arguments.

      For example:

       `./setCompartmentId.sh ocid1.compartment.oc1..aaaaaaaaxbvaatfz6yourcomparmentidhere5dnzgcbivfwvsho77myfnqq us-ashburn-1`

  6.  To create an OKE cluster, return to the OCI console and open up the hamburger button in the top-left corner of the Console and go to **Developer Services > Kubernetes Clusters**.

  ![](images/27-dev-services-oke.png " ")

  7. Make sure you are in the newly created compartment and click **Create Cluster**.
     (Please use the default schema in the unlikely situation that the newly created compartment is not quickly visible on the left pickler)

  ![](images/28-create-oke.png " ")

  8. Choose **Quick Create** as it will create the new cluster along with the new network
        resources such as Virtual Cloud Network (VCN), Internet Gateway (IG), NAT
        Gateway (NAT), Regional Subnet for worker nodes, and a Regional Subnet for load
        balancers. Click **Launch Workflow**.

  ![](images/29-create-oke-wizard.png " ")


  9. Change the name of the cluster to `mtdrworkshopcluster`, accept all the other defaults, and click **Next** to review the cluster settings.


  10. Once reviewed click **Create Cluster**, and you will see the resource creation progress.

  ![](images/31-create-oke-wizard3.png " ")

  11. Close the creation window once you can.

  ![](images/32-close-cluster-create.png " ")

  12. Once launched it should usually take around 5-10 minutes for the cluster to be fully provisioned and the Cluster Status should show Active.

  ![](images/33-click-cluster-name.png " ")

  ![](images/34-copy-cluster-id.png " ")

        _There is no need to wait for the cluster to be fully provisioned at this point as we will verify cluster creation and create a kube config in order to access it in a later step._


## **STEP 4**: Create the ATP database, TODOUSER and the TODOITEM table

1. 1. Open up the hamburger menu in the top-left corner of the Console and select **Autonomous Transaction Processing**.

![](images/menu-autonomous.png " ")

2. Click on **Create Autonomous Database**.

![](images/create-autonomous.png " ")

3. Set **Compartment, Database Name and Display Name**.
   - Set the workload type to "Transaction Processing".
   - Accept the default Deployment Type "Shared Infrastructure".
![](images/ATP-config-1.png " ")

4.  Set **ADMIN password, Network Access Type and License Type**
   - Set the database ADMIN password (12 to 30 characters, at least one uppercase letter, one lowercase letter, and one number) and confirm.
    Please note the ADMIN password; it will be required later.
   - Set the Network Access type to "Allow secure access from everywhere".
   - Set the license type to "Bring Your Own License (BYOL)" (does not matter for this workshop)
   - Click on "Create Autonomous Database"

![](images/ATP-config-2.png " ")

The database creation will take a few minutes.

5. Generate the Wallet for your ATP Connectivity
   - From the Cloud console, copy the OCID of the newly created database
     ![](images/42-copy-atp-ocids2.png " ")

   - Go back into your cloud shell and verify you are in the
      `~/mtdrworkshop/setup-dev-environment` directory.

   - Copy the following command and replace $OCID by the
     copied OCID.

     ```
     <copy>./generateWallet.sh $OCID</copy>
     ```
     - Execute generateWallet.sh ocid1.autonomousdatabase.oc1.phx.abyhqlj....

      You will be requested to enter a password for wallet encryption, this is separate for the ADMIN password but you could reuse the statement.
      A wallet.zip file will be created in the current directory.

6. Create TODOUSER using sql utility in Cloud shell

   - Stay in mtdrwokshop/setup-dev-environment directory and launch
     sql with /nolog option

      ![](images/SQLCl-Cloud-Shell.png " ")

   - Point the tool at your wallet.zip file
     SQL> set cloudconfig wallet.zip

     SQL> show tns
     ![](images/Show-tns.png " ")

     - Connect to mtdrdb_tp service, as database ADMIN user (remember the
       password given to ADMIN above)
       
      SQL> connect ADMIN@mtdrdb_tp

      - Create TODOUSER (replace <password> by a strong password)
        ```
        <copy> CREATE USER todouser IDENTIFIED BY JDBCmytodo123# DEFAULT TABLESPACE data QUOTA UNLIMITED ON data;</copy>
        ```
       - Grant some privileges to TODOUSER by executing the following command under SQLCl
      ```
      <copy>grant create session, create view, create sequence, create procedure, create table, create trigger, create type, create materialized view to todouser;</copy>
      ```
      - Connect as TODOUSER
        SQL> connect todouser@mtdrdb_tp

      - Create TODOITEM table

         Copy the following command in the Worksheet and execute.
         ```
         <copy>CREATE TABLE todoitem (
           id NUMBER GENERATED ALWAYS AS IDENTITY,
           description VARCHAR2(4000),
           creation_ts TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
           done NUMBER(1,0),
           PRIMARY KEY (id)
          );</copy>
         ```
      - Insert the first row, manually into TODOITEM table
       ```
      <copy>insert into todoitem  (description) values ('Manual item insert');</copy>
       ```
      Then commit the inserted row
      ```
      <copy>commit;</copy>
      ```

## **STEP 5**: Create an OCI Registry and Auth key
You are now going to create an Oracle Cloud Infrastructure Registry and an Auth key. The Oracle Cloud Infrastructure Registry is an Oracle-managed registry that enables you to simplify your development-to-production workflow by storing, sharing, and managing development artifacts such as Docker images.

1. Open up the hamburger menu in the top-left corner of the console and go to **Developer Services > Container Registry**.

  ![](images/21-dev-services-registry.png " ")


2. Take note of the namespace (for example, `axkcsk2aiatb` shown in the image below).

![](images/22-create-repo.png " ")

 Click **Create Repository** , specify the following details for your new repository, and click **Create Repository**.
  - Repository Name: `<tenancy name>/mtdrworkshop`
	- Access: `Public`

  Go to Cloud Shell and run `./addOCIRInfo.sh` with the namespace and repository name as arguments

  ```
  <copy>./addOCIRInfo.sh <namespace> <repository_name></copy>
  ```

  For example `./addOCIRInfo.sh axkcsk2aiatb mtdrworkshop.user1/mtdrworkshop`

3. You will now create the Auth token by going back to the User Settings page. Click the Profile icon in the top-right corner of the Console and select **User Settings**.

  ![](images/23-user-settings.png " ")

4. Click on **Auth Tokens** and select **Generate Token**.

  ![](images/24-gen-auth-token.png " ")

5. In the description type `mtdrworkshoptoken` and click **Generate Token**.

  ![](images/25-gen-auth-token2.png " ")

6. Copy the token value.

  ![](images/26-save-auth-token.png " ")

7. Go to Cloud Shell and run docker login ...
 `docker login  <USERNAME> "<AUTH_TOKEN>"` where

  * `<USERNAME>` - is the username used to log in (typically your email address). If your username is federated from Oracle Identity Cloud Service, you need to add the `oracleidentitycloudservice/` prefix to your username, for example `oracleidentitycloudservice/firstname.lastname@something.com`

  * `"<AUTH_TOKEN>"` - paste the generated token value and enclose the value in quotes.

  For example `dockerLogin.sh user.foo@bar.com "8nO[BKNU5iwasdf2xeefU;yl"`

8. Once successfully logged into Container Registry, we can list the existing docker images. Since this is the first time logging into Registry, no images will be shown.

    ```
    <copy>docker images </copy>
    ```
## **STEP 6**: Install GraalVM in Cloud Shell

We will be using JDK 11 in Cloud Shell; we will not use GraalVM Native Image

1. Run the following command

```
<copy>./installGraalVM.sh</copy>
```

## **STEP 7**: Access OKE from the Cloud Shell

1. Copy the mdtrworkshopcluster id
![](images/mtdrworkshop-cluster-id.png " ")

2. Edit mtdrworkshop/workingdir/mtdrworkshopclusterid.txt and paste the mdtrworkshopcluster id

3. Run `./verifyOKEAndCreateKubeConfig.sh`

 ```
 <copy>./verifyOKEAndCreateKubeConfig.sh</copy>
 ```

Notice `/.kube/config` is created for the cluster and the `mtdrworkshop` namespace is also created.

  ![](images/verifyOKEOutput.png " ")


Congratulations, you have completed Part I; you may now proceed to Part II.

## Acknowledgements
* **Workshop by** - Kuassi Mensah, Dir. Product Management, Java Database Access
* **Application by** - Jean de Lavarene, Sr. Director of Development, JDBC/UCP
* **Original scripts by** - Paul Parkinson, Developer Evangelist, Microservices

## Need Help?
Please submit feedback or ask for help using this [LiveLabs Support Forum](https://community.oracle.com/tech/developers/categories/building-microservices-with-oracle-converged-database). Please login using your Oracle Sign On and click the **Ask A Question** button to the left.  You can include screenshots and attach files.  Communicate directly with the authors and support contacts.  Include the *lab* and *step* in your request.
