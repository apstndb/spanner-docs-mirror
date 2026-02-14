This page describes how to grant your [Google Kubernetes Engine (GKE)](/kubernetes-engine/docs/concepts/kubernetes-engine-overview) cluster permissions to access your Spanner database.

GKE is a managed Kubernetes service that makes it easy to deploy and manage containerized applications. By using GKE and Spanner together, you can take advantage of scalability, reliability, security, and high availability in your application layer and in your database layer.

Your GKE cluster can access the Spanner API through [Workload Identity Federation for GKE](/kubernetes-engine/docs/concepts/workload-identity) . Workload Identity Federation for GKE allows a Kubernetes service account in your cluster to act as an IAM service account. The IAM service account provides [Application Default Credentials](/docs/authentication/application-default-credentials) for your pods, so that you don't need to configure each pod to use your personal user credential.

After you configure your applications to authenticate using Workload Identity Federation for GKE, you can use [Spanner client libraries](/spanner/docs/reference/libraries) to query your Spanner databases. You can also [migrate your applications to your GKE node pools](/kubernetes-engine/docs/how-to/workload-identity#migrate_applications_to) .

To create a connection in a sample environment, try the [Connecting Spanner with GKE Autopilot codelab](https://codelabs.developers.google.com/codelabs/cloud-spanner-gke-autopilot) .

## Enable Workload Identity Federation for GKE

If you haven't done so already, enable Workload Identity Federation for GKE for your GKE cluster. You can enable Workload Identity Federation for GKE on a new cluster by creating a [new node pool](/kubernetes-engine/docs/how-to/workload-identity#enable_on_cluster) or you can enable Workload Identity Federation for GKE on an [existing node pool](/kubernetes-engine/docs/how-to/workload-identity#enable-existing-cluster) . [GKE autopilot clusters](/kubernetes-engine/docs/concepts/autopilot-overview) have Workload Identity Federation for GKE enabled by default. For more information, see [Enable Workload Identity Federation for GKE](/kubernetes-engine/docs/how-to/workload-identity#enable) .

## Authenticate connection to Spanner with Workload Identity Federation for GKE

Configure your applications to authenticate to Google Cloud by using Workload Identity Federation for GKE.

1.  Ensure that your GKE Pod uses a Kubernetes `  ServiceAccount  ` object, as described in [Configure authorization and principals](/kubernetes-engine/docs/how-to/workload-identity#configure-authz-principals) .

2.  Create an IAM allow policy that grants the necessary Spanner IAM roles to the Kubernetes `  ServiceAccount  ` object. The following example grants the Spanner Database User ( `  roles/spanner.databaseUser  ` ) role:
    
    ``` text
      gcloud projects add-iam-policy-binding PROJECT_ID \
          --member="principal://iam.googleapis.com/projects/PROJECT_NUMBER/locations/global/workloadIdentityPools/PROJECT_ID.svc.id.goog/subject/ns/NAMESPACE/sa/KSA_NAME \
          --role=roles/spanner.databaseUser \
          --condition=None
    ```
    
    Replace the following:
    
      - `  PROJECT_ID  ` : The project ID of the GKE cluster.
      - `  PROJECT_NUMBER  ` : The numerical Google Cloud project number.
      - `  NAMESPACE  ` : The Kubernetes namespace that contains the ServiceAccount.
      - `  KSA_NAME  ` : The name of the ServiceAccount.

## Connect Spanner databases

After your application Pod is authenticated, you can use one of the [Spanner client libraries](/spanner/docs/reference/libraries) to query your Spanner database.

## What's next

  - Learn how to [deploy your application to the GKE cluster](/kubernetes-engine/docs/deploy-app-cluster) .
  - Learn how to [deploy your application using GKE Autopilot and Spanner](/kubernetes-engine/docs/tutorials/gke-spanner-integration) .
  - Learn more about how to [migrate existing workloads to Workload Identity Federation for GKE](/kubernetes-engine/docs/how-to/workload-identity#migrate_applications_to) .
  - Learn more about [SQL best practices](/spanner/docs/sql-best-practices) .
  - Integrate Spanner with other ORMs including [Hibernate ORM](/spanner/docs/use-hibernate) , [gorm](/spanner/docs/use-gorm) , and [Django ORM](/spanner/docs/django-orm) .
