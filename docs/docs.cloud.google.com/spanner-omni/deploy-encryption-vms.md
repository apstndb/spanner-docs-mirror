---
name: documents/docs.cloud.google.com/spanner-omni/deploy-encryption-vms
uri: https://docs.cloud.google.com/spanner-omni/deploy-encryption-vms
title: Create a deployment with TLS encryption on VMs
description: Describes how to add TLS encryption to a deployment of Spanner Omni on virtual machines (VMs).
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

This document describes how to add TLS encryption to a Spanner Omni deployment on virtual machines (VMs). A deployment with network security features uses Transport Layer Security (TLS) 1.3 to encrypt and authenticate communication within the deployment and with its clients. Spanner Omni provides mutual TLS (mTLS) for enhanced security, where both parties establish authenticity before exchanging data. mTLS is optional between the client and the server, but Spanner Omni servers communicate with each other over mTLS.

The [Preview](https://cloud.google.com/products#product-launch-stages) version of Spanner Omni doesn't support TLS encryption and stops writing data 90 days after you create a deployment. For early access to the edition with full features, [contact Google](https://cloud.google.com/consulting/spanner-omni) .

## Before you begin

Before you begin, ensure your environment meets the following requirements:

  - Ensure you have SSH access to each machine in the deployment. This access lets you download and run the Spanner Omni binary.

  - Your network must allow TCP communication on ports 15000 through 15025.

  - Each machine must have sufficient storage to host the data the deployment handles.

  - Review the [System requirements](https://docs.cloud.google.com/spanner-omni/system-requirements) page to ensure your setup meets the requirements.

  - If you run the binaries on the vSphere virtualization platform, disable TSC virtualization. To do this, add the `monitor_control.virtual_rdtsc = FALSE` setting to the virtual machine's `.vmx` configuration file.

## Step 1: Create a deployment without TLS encryption

Follow the steps in [Create a Spanner Omni VM deployment without encryption](https://docs.cloud.google.com/spanner-omni/deploy-on-kubernetes) . Verify that your VM deployment without encryption and security features works properly. This page assumes that you have created a regional deployment with three zones.

## Step 2: Generate the certificates

You need to create three sets of certificates:

| Certificate type    | Description                                                                                                        |
| ------------------- | ------------------------------------------------------------------------------------------------------------------ |
| API certificates    | API certificates help protect the Spanner API server.                                                              |
| Server certificates | Server certificates help protect inter-server communication.                                                       |
| Client certificates | End users or applications use client certificates to establish their identity and trust with Spanner Omni servers. |

A Certification Authority (CA) issues these certificates. Spanner Omni provides tools to create a CA and all three types of certificates. Perform the following steps on one of your machines.

You can create these certificates on your workstation using the Spanner Omni CLI and then transfer the certificate files to each Spanner Omni server. For more information, see the [Quickstart using the Spanner Omni CLI](https://docs.cloud.google.com/spanner-omni/cli-quickstart) .

To generate certificates, you must complete the following steps:

  - [Create a Certification Authority (CA)](https://docs.cloud.google.com/spanner-omni/deploy-encryption-vms#create-ca)
  - [Generate server certificates](https://docs.cloud.google.com/spanner-omni/deploy-encryption-vms#generate-server-certs)
  - [Generate client certificates](https://docs.cloud.google.com/spanner-omni/deploy-encryption-vms#generate-client-certs)

### Create a Certification Authority (CA)

This authority is the root CA for all client and server certificates you generate in the following steps.

    spanner certificates create-ca --ca-certificate-directory=certs

The `certs` directory contains the CA certificate. Create a copy of this certificate to use as a CA for API certificates.

    cp certs/ca.crt certs/ca-api.crt

The directory `$HOME/.spanner/private-keys` contains the private key for the CA. Back up and secure this directory. A user with access to the private key can sign arbitrary certificates that clients trusting the self-signed CA trust. Although you can use the same CA for all certificates, it is mandatory that API certificates and client certificates use the same CA. Optionally, you can create an additional CA (or use an externally trusted CA) for the API certificates. Make sure you use the right CA in the following steps while creating certificates. This document uses the same CA for all certificate types.

### Generate server certificates

You generate two types of server certificates:

  - [Spanner server certificate](https://docs.cloud.google.com/spanner-omni/deploy-encryption-vms#create-spanner-server-cert) : Encrypts communication between Spanner Omni servers.

  - [API certificate](https://docs.cloud.google.com/spanner-omni/deploy-encryption-vms#generate-api-cert) : Encrypts communication from systems interacting with the deployment.

This setup provides more flexible management of these certificates, like a certificate rotation.

#### Create the Spanner server certificate

Spanner Omni servers use server certificates to encrypt communication with each other (inter-server communication).

Create the server certificate by running the following. Replace SERVER\_LIST with a comma separated list of names of Spanner servers or suffixes.

    SERVER_NAMES=SERVER_LIST
    spanner certificates create-server --hostnames=${SERVER_NAMES} --ca-certificate-directory certs --output-directory certs

This command creates two files, `server.crt` and `server.key` , in the `certs` directory.

#### Create the API certificate

API certificates encrypt communication from systems interacting with the deployment. Using separate certificates for the API and inter-server communication lets you manage and rotate each type independently.

Create the API certificate by running the following. Replace LB\_DNS with the DNS of the load balancer.

    SERVER_NAMES=LB_DNS
    spanner certificates create-server --filename-prefix=api --hostnames=${SERVER_NAMES} --ca-certificate-directory certs --output-directory certs

This command creates two more files, `api.crt` and `api.key` , in the `certs` directory. If required, you can use an externally trusted CA for the API certificates.

#### Distribute the certificates to all servers

Copy the `certs` directory to all other servers in the deployment to start them with network security features.

    scp -r certs REMOTE_HOST:SPANNER_DIR/certs

## Step 3: Generate client certificates

You can use client certificates to authenticate users and applications in Spanner. Client certificates enable mTLS between the client and server.

Client certificates must be signed by the same CA as the API certificate and must contain a username for authorization. This example uses the `admin` user, which is the default user for each database. For more information about users, roles, and authentication options, see [Authentication and authorization in Spanner Omni](https://docs.cloud.google.com/spanner-omni/authentication) .

    USERNAME=admin
    spanner certificates create-client $USERNAME --output-directory clientcerts --ca-certificate-directory certs

This command creates `client.crt` and `client.key` files in the `clientcerts` directory. Send these files to any machine that connects to servers of the deployment.

If you plan to use the client certificates with the Java client library, you must generate the certificate key in PKCS\#8 format. Use the following command:

    USERNAME=admin
    spanner certificates create-client $USERNAME \
        --output-directory clientcerts \
        --ca-certificate-directory certs \
        --generate-pkcs8-key

## Step 4: Restart the servers

After you generate the certificates and copy them to all servers in your deployment, restart each server.

### Single server deployment

For single server deployments, run the following command:

    nohup spanner start-single-server \
        --base-dir=BASE_DIR \
        --certificate-directory=${HOME}/.spanner/certs \
        --insecure-mode=false &

The server starts. See [Step 7: Interact with the deployment](https://docs.cloud.google.com/spanner-omni/deploy-encryption-vms#interact-deployment) to interact with the deployment.

### Scale-out deployment

For scale-out deployments, start the server on each machine. The values for `server_address` and `zone` must match those in the deployment configuration. The network must resolve `server_address` . Servers use this for internal communication. Run the following to start the root server:

    nohup spanner start \
        --root \
        --server-address=HOST_NAME \
        --zone=ZONE_NAME \
        --base-dir=BASE_DIR \
        --certificate-directory=${HOME}/.spanner/certs \
        --insecure-mode=false &

The following command shows an example with specific values:

    nohup spanner start \
        --root \
        --server-address=rootserver1 \
        --zone=us-central-1a \
        --base-dir=./spanbasedir \
        --certificate-directory=${HOME}/.spanner/certs \
        --insecure-mode=false &

To enable mTLS for clients, use the `--enable-client-certificate-authentication=true` flag when starting the server.

    nohup spanner start \
        --root \
        --server-address=HOST_NAME \
        --zone=ZONE_NAME \
        --base-dir=BASE_DIR \
        --certificate-directory=${HOME}/.spanner/certs \
        --insecure-mode=false \
        --enable-client-certificate-authentication=true &

With the servers now running on each machine, you are ready to create the deployment.

## Step 5: Create a deployment with TLS encryption

Run the `spanner deployment create` command from one of the root servers to create the deployment. To enable TLS encryption, specify the base directory with the `--base-dir` flag. Ensure that you use the same BASE\_DIR that you specified when starting the root server in the previous step.

    spanner deployment create \
        --config-file=deployment.yaml \
        --base-dir=BASE_DIR

The console for each machine shows messages indicating that the deployment now includes TLS encryption. All servers communicate with each other over an encrypted channel.

## Step 6: (Optional) Configure a load balancer

To manage and distribute client traffic across the servers in your deployment, set up a load balancer. Ensure that the load balancer configuration for the health check uses HTTPS instead of HTTP. Use the following configuration details:

| Parameter          | Value                                                                                                           |
| ------------------ | --------------------------------------------------------------------------------------------------------------- |
| Protocol           | TCP                                                                                                             |
| Backend IP         | The IP addresses of your servers.                                                                               |
| Port               | `15000` (This is the default port. If you used a different port in the `--server-address` flag, use that port.) |
| Health check URL   | `https://         IP_ADDRESS        :15012/healthz`                                                             |
| Balancing strategy | roundrobin (distributes requests sequentially across servers)                                                   |

## Step 7: Interact with the deployment

You can interact with your Spanner Omni deployment from any VM using the [Spanner Omni CLI](https://docs.cloud.google.com/spanner-omni/cli-quickstart) .

You must include the following flag with each command to establish an encrypted connection:

  - `--ca-certificate-file=certs/ca-api.crt`

If you enabled mTLS for clients, also include the following flag with each command:

  - `--client-certificate-directory=clientcerts`

To sign in and interact with your deployment, follow these steps:

1.  
    
    <div id="sign-in">
    
    Sign in to Spanner Omni
    
        spanner auth login admin \
            --ca-certificate-file=certs/ca-api.crt \
            --deployment-endpoint=ENDPOINT
    
    The default password is `admin` .
    
        Successfully logged in as "admin"
    
    </div>

2.  
    
    <div id="create-db">
    
    Create a database
    
        spanner --deployment-endpoint=ENDPOINT databases create mydb --ca-certificate-file=certs/ca-api.crt
    
        Creating database...done.
    
    </div>

3.  
    
    <div id="sql-shell">
    
    Open the SQL Shell
    
        spanner sql --database=mydb --ca-certificate-file=certs/ca-api.crt
    
        Connected.
        spanner>
    
    </div>

4.  
    
    <div id="add-data">
    
    Create a table and add data
    
        spanner> create table names (nameId INT64 NOT NULL, name String(100)) Primary Key (nameId);
        Query OK, 0 rows affected (4.62 sec)
        
        spanner> insert names (nameId, name) values (1, "Jack");
        Query OK, 1 rows affected (0.18 sec)
    
    </div>

5.  
    
    <div id="verify-data">
    
    Verify the data
    
    List the databases:
    
        spanner databases list --ca-certificate-file=certs/ca-api.crt
    
        NAME  STATE  VERSION_RETENTION_PERIOD  EARLIEST_VERSION_TIME  KMS_KEY_NAME  ENABLE_DROP_PROTECTION
        mydb  READY  1h                        2025-02-07T12:25:30Z                 false
    
    Get the data from the table:
    
        spanner sql --database=mydb --ca-certificate-file=certs/ca-api.crt
    
        Connected.
        spanner> show tables;
        +----------------+
        | Tables_in_mydb |
        +----------------+
        | names          |
        +----------------+
        1 rows in set (0.14 sec)
        
        spanner> select * from names;
        +--------+--------+
        | nameId | name   |
        +--------+--------+
        | 1      | Jack   |
        +--------+--------+
        1 rows in set (18.69 msecs)
    
    </div>

## Step 8: (Optional) Scale the deployment

You can add non-root servers to a zone to scale the capacity of the zone. To do this, generate the server certificate for the non-root servers as [Step 2: Generate the certificates](https://docs.cloud.google.com/spanner-omni/deploy-encryption-vms#generate-certificates) explains, and start the server with the following command:

    spanner start \
        --server-address=NON_ROOT_MACHINE \
        --join-servers=ROOT_SERVER1,ROOT_SERVER2,ROOT_SERVER3 \
        --zone=us-central1-a \
        --base-dir=./spandir \
        --certificate-directory=${HOME}/.spanner/certs \
        --insecure-mode=false

## Next steps

  - Use [Client libraries and JDBC drivers](https://docs.cloud.google.com/spanner-omni/jdbc-driver) to connect your application with the deployment.
  - [Manage users and roles](https://docs.cloud.google.com/spanner-omni/authentication) .
