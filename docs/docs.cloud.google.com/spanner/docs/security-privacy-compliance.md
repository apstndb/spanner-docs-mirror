This document provides an overview of various controls that support the security of Spanner on Google Cloud and links to further information on how to configure the controls. Security controls such as network security options, policies, and access management can help you address your business risks and meet the privacy and regulatory requirements that apply to your business.

The security, privacy, risk, and compliance for Spanner use a [shared responsibility](https://docs.cloud.google.com/architecture/framework/security/shared-responsibility-shared-fate) model. For example, Google secures the infrastructure that Spanner and other Google Cloud services run on, and provides you with the capabilities that help you manage access to your services and resources. For more information about how we secure the infrastructure, see the [Google infrastructure security design overview](https://docs.cloud.google.com/docs/security/infrastructure/design) .

## Provisioned services

When you get started with Spanner, you enable the following APIs:

  - [`  https://spanner.googleapis.com  `](https://docs.cloud.google.com/spanner/docs/reference/rest?rep_location=global)

For more information, see [Quickstart: Create and query a database in Spanner using the Google Cloud console](https://docs.cloud.google.com/spanner/docs/quickstart-console) .

## Authentication for Google Cloud management

Administrators and developers who create and manage Spanner instances must authenticate to Google Cloud to verify their identity and access privileges. You must set up each user with a [user account that is managed](https://docs.cloud.google.com/architecture/identity/overview-google-authentication#google_for_organizations) by Cloud Identity, Google Workspace, or an [identity provider that you've federated](https://docs.cloud.google.com/architecture/identity/best-practices-for-federating) with Cloud Identity or Google Workspace. For more information, see [Overview of identity and access management](https://docs.cloud.google.com/architecture/identity/overview-google-authentication) .

After you create the user accounts, implement security best practices such as [single sign-on](https://docs.cloud.google.com/architecture/identity/single-sign-on) and [2-step verification](https://support.google.com/a/answer/175197) .

## Authentication for Google Cloud resource access

[Workforce Identity Federation](https://docs.cloud.google.com/iam/docs/workforce-identity-federation) lets you use your external identity provider (IdP) to authenticate your workforce users so that they can access Google Cloud resources. Use Workforce Identity Federation when users require programmatic access to your Google Cloud resources and you store your users' credentials in IdPs that support OpenID Connect (OIDC) or Security Assertion Markup Language (SAML).

[Workload Identity Federation](https://docs.cloud.google.com/iam/docs/workload-identity-federation) lets you use your external IdP to grant your on-premises or multi-cloud workloads access to Google Cloud resources, without using a service account key. You can use identity federation with IdPs such as Amazon Web Services (AWS), Microsoft Entra ID, GitHub, or Okta.

For more information about Spanner support for identity federation, see [Federated identity supported services](https://docs.cloud.google.com/iam/docs/federated-identity-supported-services) .

For more information about authentication in Google Cloud, see [Authentication](https://docs.cloud.google.com/iam/docs/authentication) .

## Identity and Access Management

To manage Identity and Access Management (IAM) roles at scale for your administrators and developers, consider creating separate [functional groups](https://docs.cloud.google.com/architecture/identity/overview-google-authentication#group) for your various user roles and applications. Grant the IAM roles or permissions that are required to manage Spanner to your groups. When you assign roles to your groups, follow the principle of least privilege and other [IAM security best practices](https://docs.cloud.google.com/iam/docs/using-iam-securely) . For more information, see [Best practices for using Google Groups](https://docs.cloud.google.com/iam/docs/groups-best-practices) .

For more information about setting up IAM, see [IAM overview](https://docs.cloud.google.com/iam/docs/overview) .

Spanner supports IAM fine-grained access control, which lets you use IAM principals to control access to Spanner databases, tables, columns, and views. For more information, see [About fine-grained access control](https://docs.cloud.google.com/spanner/docs/fgac-about) .

## Spanner service accounts

When you enable Spanner, Google creates service accounts for you. A [service account](https://docs.cloud.google.com/iam/docs/service-account-overview) is a special type of non-interactive Google Account that's typically used by an application or compute workload, such as a Compute Engine instance, rather than a person. Applications use service accounts to access Google APIs.

### Service agents

To enable Spanner to access your resources on your behalf, Google Cloud creates a special service account known as a [service agent](https://docs.cloud.google.com/iam/docs/service-account-types#service-agents) .

When you enable Spanner, the following Spanner service agent is created:

    service-PROJECT_ID@gcp-sa-spanner.iam.gserviceaccount.com

## Policies for Spanner

The predefined organization policies that apply to Spanner include the following:

  - Limit the creation of instances that use Spanner editions ( `  constraints/spanner.managed.restrictCloudSpannerEditions  ` )

For more information about policies, see [Use organization policies for Spanner](https://docs.cloud.google.com/spanner/docs/spanner-custom-constraints) .

You can use custom organization policies to configure restrictions on Spanner at a project, folder, or organization level. For more information, see [Creating and managing custom constraints](https://docs.cloud.google.com/resource-manager/docs/organization-policy/creating-managing-custom-constraints) .

## Network security

By default, Google applies default protections to data in transit for all Google Cloud services, including Spanner instances that are running on Google Cloud. For more information about default network protections, see [Encryption in transit](https://docs.cloud.google.com/docs/security/encryption-in-transit) .

If required by your organization, you can configure additional security controls to further protect traffic on the Google Cloud network and traffic between the Google Cloud network and your corporate network. Consider the following:

  - Spanner supports [VPC Service Controls](https://docs.cloud.google.com/vpc-service-controls/docs/overview) . VPC Service Controls let you control the movement of data in Google services and set up context-based perimeter security.
  - In Google Cloud, consider using [Shared VPC](https://docs.cloud.google.com/vpc/docs/shared-vpc) as your network topology. Shared VPC provides centralized network configuration management while maintaining separation of environments.
  - Use Cloud VPN or Cloud Interconnect to maximize security and reliability for the connection between your corporate network and Google Cloud. For more information, see [Choosing a Network Connectivity product](https://docs.cloud.google.com/network-connectivity/docs/how-to/choose-product) .

For more information about network security best practices, see [Implement zero trust](https://docs.cloud.google.com/architecture/framework/security/implement-zero-trust) and [Decide the network design for your Google Cloud landing zone](https://docs.cloud.google.com/architecture/landing-zones/decide-network-design) .

## Application connectivity

You can secure the connection between applications and Spanner using the following methods:

  - [Serverless VPC Access](https://docs.cloud.google.com/vpc/docs/serverless-vpc-access) to connect Spanner directly with Cloud Run.
  - [Private Service Connect](https://docs.cloud.google.com/vpc/docs/private-service-connect) to connect to a managed application in another VPC on Google Cloud using Spanner private IP address. Use this method to keep traffic in Google Cloud.

For more information about options for setting up connections to services without an external IP address, see [Private access options for services](https://docs.cloud.google.com/vpc/docs/private-access-options) .

## Client authentication

Spanner provides the following authentication methods for clients:

  - [IAM authentication](https://docs.cloud.google.com/spanner/docs/iam)

## Data protection and privacy

Spanner encrypts your data that is stored in Google Cloud using [default encryption](https://docs.cloud.google.com/docs/security/encryption/default-encryption) . Example data includes the following:

  - Table and column names
  - Index names
  - Database schema
  - Data stored in tables

This data can only be accessed by Spanner instances.

You can enable [customer-managed encryption keys (CMEK)](https://docs.cloud.google.com/spanner/docs/cmek) to encrypt your data at rest. With CMEK, keys are stored in Cloud Key Management Service (Cloud KMS) as software-protected keys or hardware-protected keys with Cloud HSM, but they are managed by you. To provision encryption keys automatically, you can enable [Cloud KMS Autokey](https://docs.cloud.google.com/kms/docs/autokey-overview) . When you enable Autokey, a developer can request a key from Cloud KMS, and the service agent provisions a key that matches the developer's intent. With Cloud KMS Autokey, keys are available on demand, are consistent, and follow industry-standard practices.

In addition, Spanner supports Cloud External Key Manager (Cloud EKM), which lets you store your keys in an external key manager outside of Google Cloud. For more information, see [Customer-managed encryption keys (CMEK) overview](https://docs.cloud.google.com/spanner/docs/cmek) .

### Where data is processed

Spanner supports [data residency](https://cloud.google.com/terms/data-residency) for data that is stored on Google Cloud. Data residency lets you choose the regions that you want your data to be stored in using the [Resource Location Restriction policy constraint](https://docs.cloud.google.com/resource-manager/docs/organization-policy/defining-locations) . You can use [Cloud Asset Inventory](https://docs.cloud.google.com/asset-inventory/docs/asset-inventory-overview) to verify the location of Spanner resources.

If you require data residency for data in use, you can configure Assured Workloads. For more information, see [Assured Workloads and data residency](https://docs.cloud.google.com/assured-workloads/docs/data-residency) .

### Data privacy

To help protect the privacy of your data, Spanner conforms to the [Common Privacy Principles](https://cloud.google.com/privacy/common-privacy-principles) .

Spanner acts as a data processor for Customer Data. Google also acts as a data controller for information such as billing and account management and abuse detections. For more information, see [Google Cloud Privacy Notice](https://cloud.google.com/terms/cloud-privacy-notice) .

## Audit logging

Spanner writes the following types of audit logs:

  - **Admin Activity audit logs** : Includes `  ADMIN WRITE  ` operations that write metadata or configuration information.

  - **Data Access audit logs** : Includes `  ADMIN READ  ` operations that read metadata or configuration information. Also includes `  DATA READ  ` and `  DATA WRITE  ` operations that read or write user-provided data.

  - **System Event audit logs** : Identifies automated Google Cloud actions that modify the configuration of resources.

For more information, see [Audit logging](https://docs.cloud.google.com/spanner/docs/audit-logging) .

## Access transparency

You can use [Access Approval](https://docs.cloud.google.com/assured-workloads/access-approval/docs/overview) and [Access Transparency](https://docs.cloud.google.com/assured-workloads/access-transparency/docs/overview) to control access to Spanner instances by Google personnel who support the service. Access Approval lets you approve or dismiss requests for access by Google employees. Access Transparency logs offer near real-time insight when Google Cloud administrators access the resources.

## Monitoring and incident response

You can use a variety of tools to help you monitor the performance and security of Spanner. Consider the following:

  - Logs Explorer to view and analyze event logs and create [custom metrics](https://docs.cloud.google.com/logging/docs/logs-based-metrics) and alerts.
  - Use the Cloud Monitoring dashboard to monitor the performance of Spanner. For more information, see [Monitor Spanner](https://docs.cloud.google.com/spanner/docs/monitoring-cloud) .
  - Deploy [cloud controls and frameworks](https://docs.cloud.google.com/security-command-center/docs/compliance-manager-apply-framework) in Security Command Center to detect vulnerabilities and threats to Spanner (such as privilege escalations). You can set up alerts and [playbooks](https://docs.cloud.google.com/security-command-center/docs/playbooks-overview) for your security operations center (SOC) analysts so that they can respond to findings.

## Certifications and compliance

Meeting your regulatory requirements is a [shared responsibility](https://docs.cloud.google.com/architecture/framework/security/shared-responsibility-shared-fate) between you and Google.

Spanner has received a variety of certifications, including the following:

  - ISO 27001
  - SOC 2
  - HIPAA

For more information about Google Cloud compliance with different regulatory frameworks and certifications, see the [compliance resource center](https://cloud.google.com/security/compliance) .

Spanner also supports Assured Workloads, which lets you apply controls to specific folders in your Google organization that support regulatory, regional, or sovereign requirements. For more information, see [Supported products by control package](https://docs.cloud.google.com/assured-workloads/docs/supported-products) .

## What's next

  - [Enable backups](https://docs.cloud.google.com/spanner/docs/backup) .
  - Use [Google Threat Intelligence](https://cloud.google.com/security/products/threat-intelligence) to track external threats that apply to your business.
  - Learn more about [Identity and Access Management](https://docs.cloud.google.com/spanner/docs/iam) in Spanner.
  - Learn more about [data encryption](https://docs.cloud.google.com/spanner/docs/cmek) in Spanner.
