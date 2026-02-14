This page explains how you can search for and manage your Spanner resources by using Data Catalog.

**Caution:** Data Catalog is [deprecated](/data-catalog/docs/deprecations) in favor of [Dataplex Universal Catalog](/dataplex/docs/catalog-overview) . Dataplex Universal Catalog is also integrated with Spanner, offering similar capabilities. You can use Dataplex Universal Catalog to enrich your data with aspects, which are the equivalent of Data Catalog tags. For more information, see [Manage aspects and enrich metadata](/dataplex/docs/enrich-entries-metadata) .

Data Catalog is a fully managed, scalable metadata management service within Dataplex Universal Catalog. It automatically catalogs the following metadata about Spanner instances, databases, tables, columns, and views:

  - Name and fully-qualified name
  - Location (region)
  - Creation date and last modification date
  - Schema (for tables and views)
  - Description

Spanner metadata is automatically synced to Data Catalog at regular intervals, typically every few hours. You can use Data Catalog to discover and understand your Spanner metadata. Use Data Catalog to aid with the following activities:

  - Analysis, including dependencies and suitability for a use case
  - Change management
  - Data movement (pipelines)
  - Schema evolution

With Data Catalog, you can curate metadata by attaching tags to Spanner metadata entries. Each tag can have multiple metadata fields, and can be based on a predefined or custom tag template.

For example, you could attach the following tag to a column that contains a social security number, which is personal identifiable information (PII):

``` text
pii:true
pii_type:SSN
```

When you [move an instance](/spanner/docs/move-instance) that uses tags, the tags aren't automatically moved to the destination instance. Instead, you need to export tags from the source instance before moving the instance, and import the tags into the destination instance. For more information, see [Export and import tags](#import-export-tags) .

To learn more about Data Catalog see [What is Data Catalog](/data-catalog/docs/concepts/overview) .

**Note:** Data Catalog refers to the resources in Spanner and in other Google Cloud services as *assets* . In this page, we refer to Spanner resources—instances, databases, tables, and views—as Spanner assets. We also use the term *assets* to refer to both the resources and Data Catalog metadata for the resources.

## Before you begin

1.  Sign in to your Google Cloud account. If you're new to Google Cloud, [create an account](https://console.cloud.google.com/freetrial) to evaluate how our products perform in real-world scenarios. New customers also get $300 in free credits to run, test, and deploy workloads.

2.  In the Google Cloud console, on the project selector page, select or create a Google Cloud project.
    
    **Roles required to select or create a project**
    
      - **Select a project** : Selecting a project doesn't require a specific IAM role—you can select any project that you've been granted a role on.
      - **Create a project** : To create a project, you need the Project Creator role ( `  roles/resourcemanager.projectCreator  ` ), which contains the `  resourcemanager.projects.create  ` permission. [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .
    
    **Note** : If you don't plan to keep the resources that you create in this procedure, create a project instead of selecting an existing project. After you finish these steps, you can delete the project, removing all resources associated with the project.

3.  [Verify that billing is enabled for your Google Cloud project](/billing/docs/how-to/verify-billing-enabled#confirm_billing_is_enabled_on_a_project) .

4.  In the Google Cloud console, on the project selector page, select or create a Google Cloud project.
    
    **Roles required to select or create a project**
    
      - **Select a project** : Selecting a project doesn't require a specific IAM role—you can select any project that you've been granted a role on.
      - **Create a project** : To create a project, you need the Project Creator role ( `  roles/resourcemanager.projectCreator  ` ), which contains the `  resourcemanager.projects.create  ` permission. [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .
    
    **Note** : If you don't plan to keep the resources that you create in this procedure, create a project instead of selecting an existing project. After you finish these steps, you can delete the project, removing all resources associated with the project.

5.  [Verify that billing is enabled for your Google Cloud project](/billing/docs/how-to/verify-billing-enabled#confirm_billing_is_enabled_on_a_project) .

6.  Enable the Data Catalog API for the project.

7.  Check permissions.
    
    You need certain Identity and Access Management (IAM) roles and permissions to search for and attach tags to Spanner assets. For details, see [Required IAM roles and permissions for Data Catalog](#iam-permissions) .

## Create tag templates

Tag temples are reusable structures that you can use to rapidly create new tags. Templates help you avoid duplication of work and incomplete tags. Create as many tag templates as you need.

To learn more, see [Tags and tag templates](/data-catalog/docs/tags-and-tag-templates) .

## Attach tags to Spanner assets

Attaching tags to Spanner assets lets you do the following:

  - Add business metadata to the assets.
  - Search for assets by business metadata and other custom metadata.

To learn more, see [Tags and tag templates](/data-catalog/docs/tags-and-tag-templates) .

## Search for Spanner assets

Use the Dataplex Universal Catalog search page in the Google Cloud console to search for Spanner assets.

1.  Go to the Dataplex Universal Catalog search page.

2.  In the **Filters panel** , under **Systems** , select **Spanner** .
    
    Dataplex Universal Catalog displays all known Spanner assets.

3.  Optional: Do one of the following to narrow down the search:
    
      - Use the faceted search on the **Search** page. Select checkboxes under **Data types** , **Projects** , and **Tags** .
    
      - In the search field, append a search parameter after `  system=cloud_spanner  ` . Separate parameters with spaces.
    
    For example, to view only databases, enter the following text in the search field, and then press `  Enter  ` .
    
    ``` text
     system=cloud_spanner type=database
    ```
    
    **Note:** To search for Spanner instances, use `  type=service  ` . To search for an instance configuration, use `  instance_config=configuration-name  ` .
    
    You can also use parentheses and the logical operators `  and  ` and `  or  ` for complex expressions. To learn more about the expressions that you can use in the search field, see [Data Catalog search syntax](/data-catalog/docs/how-to/search-reference) .

4.  In the results table, click the name of an asset to view the metadata for that asset.

5.  Optional: Do any of the following:
    
      - Click **ADD OVERVIEW** to add a rich text description of the asset.
      - Click **ATTACH TAGS** to add a tag to the asset.
      - For a table, click the **SCHEMA** tab to view the table columns.
      - For an instance (SERVICE), to view member databases, click the **ENTRY LIST** tab, and then click **VIEW CHILD ENTRIES IN SEARCH** . (If the **ENTRY LIST** tab doesn't appear, then the instance has no databases.)

## Example workflow - Drill down from instance to columns

In this example workflow, you start by searching for a Spanner instance, then view a member database, then view a table in that database, and then view the columns in the table.

1.  Go to the Dataplex Universal Catalog search page.

2.  In the **Filters panel** , under **Systems** , select **Spanner** .

3.  To view all Spanner instances in Data Catalog, either select the **Service** checkbox under **Data types** , or enter the following text in the search field and press `  Enter  ` .
    
    ``` text
    system=cloud_spanner type=service
    ```

4.  Select an instance name.

5.  On the **Spanner service details** page, click the **ENTRY LIST** tab, and then click **VIEW CHILD ENTRIES IN SEARCH** .
    
    Dataplex Universal Catalog displays the databases in the instance.
    
    **Note:** If there is no **ENTRY LIST** tab, return to the **Search** page and choose a different instance.

6.  On the **Spanner database details** page, click the **ENTRY LIST** tab, and then click **VIEW CHILD ENTRIES IN SEARCH** .
    
    Dataplex Universal Catalog displays the tables in the database.

7.  Select a table name, and then on the **Spanner table details** page, click **SCHEMA** to see the table columns.

8.  Optional: To add a tag to a column, click the plus sign under **Column tags** .

**Note:** This workflow demonstrates drilling down from an instance to a table. You can go directly to a list of tables by using the **Filters** panel or by entering `  system=cloud_spanner,type=table  ` in the search field.

## Export and import tags

When you [move a Spanner instance](/spanner/docs/move-instance) , the moving process deletes the instance tags that you created in Data Catalog. To preserve your tags, you need to do the following:

  - Query the tags associated with the instance.
  - Copy the details for the tags.
  - Create the tags on the moved instance.

Spanner sync data every 6 hours. Any metadata changes made on Spanner assets like instances, databases, tables, views or columns could take approximately 6 hours to propagate to Data Catalog.

### Export tags from the source instance configuration

To list the tags for an instance (entry or entry group), use the Google Cloud CLI [`  gcloud data-catalog tags list  `](/sdk/gcloud/reference/data-catalog/tags/list) command as follows:

``` text
curl \
'https://datacatalog.googleapis.com/v1/projects/PROJECT/locations/LOCATION/entryGroups/ENTRY_GROUP/tags?key=API_KEY' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Accept: application/json' \
  --compressed
```

Replace the following:

  - PROJECT : Project that contains the tags.
  - LOCATION : Location for the tags.
  - API\_KEY : A unique string that lets you access an API.
  - ACCESS\_TOKEN : The access token that your application uses to authenticate to the service.

### Import tags into the destination configuration

Before you complete this procedure, do the following:

  - [Move the instance](/spanner/docs/move-instance) .
  - [Update Data Catalog with metadata](/sdk/gcloud/reference/data-catalog/entries/update) .

To copy over a tag, create the tags on the moved instance using the [`  gcloud data-catalog tags create  `](/sdk/gcloud/reference/data-catalog/tags/create) command as follows:

``` text
curl --request POST \
'https://datacatalog.googleapis.com/v1/entries:lookup?fullyQualifiedName=FQN&location=LOCATION&project=PROJECT&key=API_KEY' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Accept: application/json' \
  --header 'Content-Type: application/json' \
  --data '{"column":"myColumnName","name":"myTagName","template":"myTemplateName","fields":{"myDoubleField":{"doubleValue":0}}}' \
  --compressed
```

Replace the following:

  - FQN : Fully qualified name (FQN) of the resource. FQNs take two forms:
    
    For non-regionalized resources: {SYSTEM}:{PROJECT}.{PATH\_TO\_RESOURCE\_SEPARATED\_WITH\_DOTS}
    
    For regionalized resources: {SYSTEM}:{PROJECT}.{LOCATION\_ID}.{PATH\_TO\_RESOURCE\_SEPARATED\_WITH\_DOTS}
    
    Example for a DPMS table:
    
    dataproc\_metastore:{PROJECT\_ID}.{LOCATION\_ID}.{INSTANCE\_ID}.{DATABASE\_ID}.{TABLE\_ID}

  - LOCATION : Location where the lookup is performed.

  - PROJECT : Project where the lookup is performed.

  - API\_KEY : A unique string that lets you access an API.

  - ACCESS\_TOKEN : The access token that your application uses to authenticate to the service.

## Required IAM roles and permissions for Data Catalog

The following table shows the required IAM roles and permissions for the various Data Catalog operations.

**Note:** Data Catalog doesn't support Spanner fine-grained access control.

**Data Catalog operation**

**Spanner resource**

**Roles or permissions required**

Create a tag template

N/A

roles/datacatalog.tagTemplateCreator

Search for Spanner resources

Instance

spanner.instances.get

Database

spanner.databases.get

Table

spanner.databases.get

Views

spanner.databases.get

View public tags

Instance

spanner.instances.get

Database

spanner.databases.get

Table

spanner.databases.get

Views

spanner.databases.get

View private tags

Instances

datacatalog.tagTemplates.getTag + spanner.instances.get

Databases

datacatalog.tagTemplates.getTag + spanner.databases.get

Tables

datacatalog.tagTemplates.getTag + spanner.databases.get

Views

datacatalog.tagTemplates.getTag + spanner.databases.get

Attach a tag to a Spanner resource using a tag template

Instances

datacatalog.tagTemplates.use + spanner.instances.updateTag

Databases

datacatalog.tagTemplates.use + spanner.databases.updateTag

Tables

datacatalog.tagTemplates.use + spanner.databases.updateTag

Views

datacatalog.tagTemplates.use + spanner.databases.updateTag

The `  spanner.instances.UpdateTag  ` permission is included in the following role:

  - roles/spanner.admin

The `  spanner.databases.UpdateTag  ` permission is included in the following roles:

  - roles/spanner.admin
  - roles/spanner.databaseAdmin
  - roles/spanner.databaseUser

For more information, see [Predefined roles](/spanner/docs/iam#roles) .

## What's next

  - [What is Data Catalog](/data-catalog/docs/concepts/overview)
  - [Roles to search Google Cloud resources](/data-catalog/docs/concepts/iam#roles_to_search_resources)
  - [About fine-grained access control](/spanner/docs/fgac-about)
