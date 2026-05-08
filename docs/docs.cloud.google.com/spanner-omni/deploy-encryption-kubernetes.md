> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

Spanner Omni uses TLS 1.3 to encrypt data that flows between the client and server, and between Spanner Omni servers. Spanner Omni provides mTLS for enhanced security where both parties establish the authenticity of each other before exchanging any data. If you use encryption, then your servers must communicate over mTLS. You can choose whether your client and server also use mTLS.

The [Preview](https://cloud.google.com/products#product-launch-stages) version of Spanner Omni doesn't support TLS encryption and stops writing data 90 days after you create a deployment. For early access to the edition with full features, [contact Google](https://cloud.google.com/consulting/spanner-omni) .

## Before you begin

Ensure that you meet the following requirements:

  - Create a Kubernetes cluster. The configuration supports Google Kubernetes Engine (GKE) and Amazon Elastic Kubernetes Service (Amazon EKS). You might need to customize the configuration to work in other environments.

  - Ensure the Kubernetes cluster can access the Artifact Registry artifact that hosts the Spanner Omni container.

  - Install and configure the [`kubectl` command line tool](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) and [Helm](https://helm.sh/) .

  - If you set up the Kubernetes environment on vSphere virtualization platform machines, disable Time Stamp Counter (TSC) virtualization by adding `monitor_control.virtual_rdtsc = FALSE` to the virtual machine's `.vmx` configuration file. This ensures TrueTime works correctly.

  - Verify your environment meets [Spanner Omni system requirements](https://docs.cloud.google.com/spanner-omni/system-requirements) .

## Step 1: Generate the certificates

You must create three sets of certificates:

  - **API certificates** : These help protect the Spanner Omni API server.

  - **Server certificates** : These help protect inter-server communication.

  - **Client certificates** : End users or applications use these to establish their identity and trust with Spanner Omni servers.

A Certification Authority (CA) issues these certificates. Spanner Omni provides tools to create a self-signed CA and all three types of certificates.

Perform the following steps on one of your machines. The steps assume that the namespace is `spanner-ns` . Change this to the namespace that you intend to use in your deployment.

You can create these certificates on your workstation using the Spanner Omni CLI.

### 1\. Create a Certification Authority (CA)

A Certification Authority (CA) issues all certificates. Your organization might have a central CA, or you can use a public CA. While you can use the same CA for all certificates, API certificates and client certificates must use the same CA.

Spanner Omni lets you create a private CA.

    ./google/spanner/bin/spanner certificates create-ca --ca-certificate-directory=certs

The `create-ca` command generates the CA certificate in the `certs` directory. You can either copy this certificate to use as the CA for API certificates, or create a different CA. Ensure that you use the correct CA when creating certificates.

    cp certs/ca.crt certs/ca-api.crt

The directory `$HOME/.spanner/private-keys` contains the private key for the CA. It's critical to back up and secure this directory. Users with access to the private key can sign arbitrary certificates that clients trusting the self-signed CA trust. Optionally, you can create an additional CA (or use an externally trusted CA) for the API certificates. This document uses the same CA for all types of certificates.

### 2\. Generate server certificates

You must generate two types of server certificates:

  - **API certificate** : Use this certificate to encrypt communication from systems interacting with the deployment.

  - **Spanner server certificate** : Spanner Omni servers use this certificate to encrypt communication with each other.

This configuration provides flexibility in managing these certificates. For example, it lets you use certificate rotation.

#### Create the Spanner server certificate

To create the server certificate, run the following command:

    # Comma-separate names of the Spanner servers; wildcards are supported.
    SERVER_NAMES=*.pod.NAMESPACE
    ./google/spanner/bin/spanner certificates create-server --hostnames=${SERVER_NAMES} --ca-certificate-directory certs --output-directory certs

This command creates `server.crt` and `server.key` in the `certs` directory.

#### Create the API certificate

To create the API certificate, run the following command:

    OMNI_ENDPOINT=spanner.NAMESPACE
    ./google/spanner/bin/spanner certificates create-server --filename-prefix=api --hostnames=${OMNI_ENDPOINT} --ca-certificate-directory certs --output-directory certs

This command creates `api.crt` and `api.key` in the `certs` directory. If necessary, use an externally trusted CA for the API certificates.

### 3\. Generate client certificates

You can use client certificates to authenticate users and applications. Client certificates enable mTLS between the client and the server. You can skip this step if you don't plan to use mTLS.

The same CA that signs the API certificate must sign client certificates, which must also contain a username for authorization. For this example, use the `admin` user, which is the default user for each new database. For more information, see [Authentication and authorization in Spanner Omni](https://docs.cloud.google.com/spanner-omni/authentication) .

    USERNAME=admin
    ./google/spanner/bin/spanner certificates create-client $USERNAME --output-directory clientcerts --ca-certificate-directory certs

This command creates `client.crt` and `client.key` in the `clientcerts` directory. Send these files to any machine that connects to the deployment.

If you plan to use client certificates with the Java client library, you must generate the certificate key in PKCS\#8 format. Use the following command:

    USERNAME=admin
    ./google/spanner/bin/spanner certificates create-client $USERNAME --output-directory clientcerts --ca-certificate-directory certs --generate-pkcs8-key

## Step 2: Push the certificates to the Kubernetes cluster

Run the following commands to push the certificates to your Kubernetes cluster:

    kubectl create namespace NAMESPACE
    
    kubectl create secret generic tls-certs \
      --from-file=ca.crt="certs/ca.crt" \
      --from-file=ca-api.crt="certs/ca-api.crt" \
      --from-file=server.crt="certs/server.crt" \
      --from-file=server.key="certs/server.key" \
      --from-file=api.crt="certs/api.crt" \
      --from-file=api.key="certs/api.key" \
      -n NAMESPACE

## Step 3: Create the deployment with TLS encryption

Follow these steps to create your deployment with TLS encryption.

### 1\. Prepare the Helm configuration

Refer to [Create a Helm chart configuration](https://docs.cloud.google.com/spanner-omni/create-helm-configuration) and create the deployment configuration for your environment.

To enable TLS, set the following values in your Helm chart configuration:

    # Enables TLS
    global:
      insecureMode: false
    
    # Enables client certificate authentication (mTLS)
    deployment:
      enableClientCertificateAuthentication: true

### 2\. Create the deployment

Run the following command to create the deployment:

    kubectl create ns monitoring
    
    helm upgrade --install spanner-omni oci://us-docker.pkg.dev/spanner-omni/charts/spanner-omni \
      --version VERSION \
      --set global.platform=gke \
      --set global.insecureMode=false \
      --set deployment.enableClientCertificateAuthentication=true \
      --namespace NAMESPACE \
      --set monitoring.enabled=true

The command triggers a bootstrap job. You can track the progress by watching the logs of this job:

    kubectl logs -n NAMESPACE -l app.kubernetes.io/component=bootstrap -f

The output indicates the progress. When complete, you see a message stating "Deployment created successfully".

### 3\. Check the status of the pods

Run the following command to verify the pod status:

    kubectl get pods --watch --namespace NAMESPACE

All pods are in the `READY` state.

### 4\. Update the certificate and deployment with load balancer details

This step is required if you want clients to connect from outside the Kubernetes cluster.

    # Get the service details
    kubectl get service spanner -n NAMESPACE
    
    # The EXTERNAL-IP:PORT is the API or deployment endpoint for your deployment.
    # Update the API certificate with these details.
    OMNI_ENDPOINT=EXTERNAL_IP,spanner.NAMESPACE.svc
    ./google/spanner/bin/spanner certificates update --filename_prefix=api --hostnames=${OMNI_ENDPOINT} --ca_certificate_directory certs --output_directory certs --overwrite
    
    # Update the secrets in Kubernetes
    kubectl patch secret tls-certs -n NAMESPACE -p "{\"data\":{\"api.crt\":\"$(base64 -w 0 certs/api.crt)\"}}"

## Step 4: Interact with Spanner Omni

You can interact with your Spanner Omni deployment from any VM using the Spanner Omni CLI.

If you enabled mTLS for clients, use the following flags with each command:

  - ` --client-certificate-directory= CLIENT_CERTIFICATE_DIRECTORY  `

  - ` --ca-certificate-file= API_CA_CERT_FILE_PATH  `

### 1\. Log in to Spanner Omni

Run the following command to log in:

    ./google/spanner/bin/spanner auth login admin --ca-certificate-file=certs/ca-api.crt \
    --client-certificate_directory=clientcerts --deployment-endpoint=DEPLOYMENT_ENDPOINT

The default password is `admin` .

    Successfully logged in as "admin"

### 2\. Create a database

Run the following command to create a database:

    ./google/spanner/bin/spanner --deployment-endpoint=DEPLOYMENT_ENDPOINT databases create DATABASE_NAME --ca-certificate-file=certs/ca-api.crt --client-certificate-directory=clientcerts

### 3\. Open the SQL Shell

Run the following command to open the shell:

    ./google/spanner/bin/spanner sql --database=DATABASE_NAME --deployment-endpoint=DEPLOYMENT_ENDPOINT --ca-certificate-file=certs/ca-api.crt --client-certificate-directory=clientcerts

### 4\. Create a table and add data

Run the following SQL commands:

    spanner> CREATE TABLE names (nameId INT64 NOT NULL, name String(100)) PRIMARY KEY (nameId);
    Query OK, 0 rows affected (4.62 sec)
    
    spanner> INSERT names (nameId, name) VALUES (1, "Jack");
    Query OK, 1 rows affected (0.18 sec)

### 5\. Verify the deployment

To list the databases, run the following command:

    ./google/spanner/bin/spanner databases list --ca-certificate-file=certs/ca-api.crt --client-certificate-directory=clientcerts --deployment-endpoint=DEPLOYMENT_ENDPOINT

The output looks similar to the following:

| NAME           | STATE | VERSION\_RETENTION\_PERIOD | EARLIEST\_VERSION\_TIME | ENABLE\_DROP\_PROTECTION |
| -------------- | ----- | -------------------------- | ----------------------- | ------------------------ |
| DATABASE\_NAME | READY | 1h                         | 2025-02-07T12:25:30Z    | false                    |

To query the data, run the following command:

    ./google/spanner/bin/spanner sql --database=DATABASE_NAME --ca-certificate-file=certs/ca-api.crt --client-certificate-directory=clientcerts --deployment-endpoint=DEPLOYMENT_ENDPOINT

Run the following SQL commands:

    SHOW TABLES;
    SELECT * FROM names;

Alternatively, you can follow the instructions in [Using PGAdapter with Spanner Omni](https://docs.cloud.google.com/spanner-omni/pgadapter) to configure PGAdapter and interact using tools like `psql` .

## Step 5: Monitor the deployment

If you installed Spanner Omni with `monitoring.enabled=true` , Prometheus scrapes metrics. You can use Grafana to visualize these metrics.

### 1\. Get the service details

Run the following commands to get the service details:

    # Prometheus service details. Default port is 9090.
    kubectl get service prometheus-service -n monitoring
    
    # Grafana service details. Default port is 3000.
    kubectl get service grafana -n monitoring

## What's next

  - Use [Client libraries and JDBC drivers](https://docs.cloud.google.com/spanner-omni/jdbc-driver) to connect your application with the deployment.
  - [Manage users and roles](https://docs.cloud.google.com/spanner-omni/authentication) .
