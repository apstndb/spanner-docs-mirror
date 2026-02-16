This page describes how you can use Google Cloud tags to manage access to your Spanner instances.

A [Google Cloud tag](/resource-manager/docs/tags/tags-overview) is a key-value pair that you can attach to your Google Cloud resources, such as projects or Spanner instances. You can use tags to group and organize your instances, and to conditionally set [Identity and Access Management (IAM) access policies](/iam/docs/policies) based on whether an instance has a specific tag. You can create and manage Spanner instance tags using the Google Cloud CLI or Google Cloud console. After you create your tags, you can create a tag binding to attach the tag to your Google Cloud resources. Tag bindings are inherited by children of the resource according to the [Google Cloud resource hierarchy](/resource-manager/docs/cloud-platform-resource-hierarchy) . For example, if you attach a tag to your project, all instances in that project inherit the tag. You can also use [labels](/resource-manager/docs/creating-managing-labels#what-are-labels) to organize your Google Cloud resources, but you can't use labels to set conditions on IAM policies.

To learn more about tags, see [Tags overview](/resource-manager/docs/tags/tags-overview) .

## Common use cases for Spanner instance tags

Some common use cases for tags include:

  - **IAM tags:** IAM roles based on whether an instance has a specific tag. The presence or absence of a tag value is the condition for that IAM policy and helps control access to your Spanner instance.

  - **State tags:** Indicate and manage the state of an instance by creating tags. For example, `  state:active  ` , `  state:todelete  ` , and `  state:archive  ` .

  - **Environment tags:** Specify production, test, and development environments for instances by creating key-value pairs such as `  env:prod  ` , `  env:dev  ` , and `  env:test  ` .

## How to create and manage Spanner instance tags

Tags are structured as key-value pairs. You create a tag key under your organization resource, and then attach tag values to the tag key (for example, a tag key `  environment  ` with values `  prod  ` and `  dev  ` ). You can then create a tag binding that links the tag value to a Google Cloud resource, such as a project or Spanner instance. Note that you cannot assign a tag to a database.

### Required permissions

The permissions you need depend on the action you need to perform. For more information, see [Required permissions](/resource-manager/docs/tags/tags-creating-and-managing#required-permissions) in the Resource Manager documentation.

### Create tag keys and values

Before you can attach a tag to your instance, you must create the tag and assign its value. To create tag keys and tag values, see [Creating a tag](/resource-manager/docs/tags/tags-creating-and-managing#creating_tag) and [Adding a tag value](/resource-manager/docs/tags/tags-creating-and-managing#adding_tag_values) .

### Attach a tag to an instance

After you create your tag key-value pairs, you can create a tag binding and attach it to your Spanner instance.

### Console

1.  In the Google Cloud console, go to the **Spanner Instances** page.

2.  Select the checkbox next to the instance for which you would like to attach a tag.

3.  Click label\_important **Tags** .

4.  If your organization doesn't appear in the **Tags** panel, click **Select scope** . Select your organization and click **Open** .

5.  In the Tags panel, select **Add tag** .

6.  In the **Key** field, select the key for the tag you want to attach from the list. You can filter the list by typing keywords.

7.  In the **Value** field, select the value for the tag you want to attach from the list. You can filter the list by typing keywords.

8.  If you want to attach more tags, click add **Add Tag** , and select the key and value for each.

9.  Click **Save** .

10. In the **Confirm** dialog, click **Confirm** to attach the tag.
    
    A notification confirms that your tags updated.

### gcloud

To create a tag binding and attach it to your instance, run the following command:

``` text
gcloud resource-manager tags bindings create
--parent=//spanner.googleapis.com/projects/PROJECT_ID/instances/INSTANCE_ID
--tag-value=TAG_VALUE_NAME
--location=LOCATION
```

  - `  PROJECT_ID  ` : The ID of the project.
  - `  INSTANCE_ID  ` : The ID of the instance.
  - `  TAG_VALUE_NAME  ` : `  TAG_VALUE_NAME  ` is the permanent ID or namespace ID of the tag value to be attached. For example: `  tagValues/4567890123  ` (permanent ID) or `  12345678/env/prod  ` (namespace ID). The namespace ID consists of the following:
      - `  ORG_ID  ` : The ID of the organization.
      - `  KEY_NAME  ` : The display (short) name of your tag key. For example, `  env  ` .
      - `  VALUE_NAME  ` : The display (short) name of your tag value. For example, `  prod  ` .
  - `  LOCATION  ` : The location of an instance varies depending on its configuration:
      - For a regional instance, the location of its configuration. For example, `  us-east1  ` .
      - For a dual-region instance, the 2-letter resource location. For example, `  de  ` for `  dual-region-germany1  ` .
      - For a multi-region instance, the name of its configuration. For example, `  eur3  ` or `  nam-eur-asia1  ` .

For example, to create a tag binding on your Spanner instance `  my-instance  ` with the tag key-value pair `  env:prod  ` , run the following command:

``` text
gcloud resource-manager tags bindings create
--parent=//spanner.googleapis.com/projects/my-project/instances/my-instance
--tag-value=123456789012/env/prod
--location=us-east1
```

### API

To create a tag binding and attach it to your Spanner instance using REST or RPC API, see [Attaching tags to resources](/resource-manager/docs/tags/tags-creating-and-managing#api_8) .

### IAM conditions and tags

You can use tags and IAM conditions to conditionally grant role bindings to users. If an IAM policy with conditional role bindings is applied, changing or deleting the tag attached to a resource might remove user access to that resource.

**Note:** When managing access for users in [external identity providers](/iam/docs/workforce-identity-federation) , replace instances of Google Account principal identifiers—like `  user:kiran@example.com  ` , `  group:support@example.com  ` , and `  domain:example.com  ` —with appropriate [Workforce Identity Federation principal identifiers](/iam/docs/principal-identifiers) .

For more information, see [Overview of IAM Conditions](/iam/docs/conditions-overview) .

### Console

To use tags to conditionally grant role bindings to users, see [Managing access to tags](/resource-manager/docs/tags/tags-creating-and-managing#managing_access) .

### gcloud

To apply a tag-based condition to an IAM policy, make sure you have the required permissions, then run the following command:

``` text
gcloud organizations add-iam-policy-binding ORG_ID
--role=roles/ROLE --member=PRINCIPAL
--condition=resource.matchTag('PROJECT_ID/KEY_NAME', 'VALUE_NAME')
```

  - `  ORG_ID  ` : The ID of the organization.
  - `  ROLE  ` : The role name to assign to the principal. The role name is the complete path of a predefined role, such as `  roles/logging.viewer  ` , or the role ID for a custom role, such as `  organizations/{ORG_ID}/roles/logging.viewer  ` .
  - `  PRINCIPAL  ` : The principal on which you want to add the role binding. This should be in the form `  user|group|serviceAccount:email  ` or `  domain:domain  ` . For example, `  user:test-user@gmail.com  ` , `  group:admins@example.com  ` , `  serviceAccount:test123@example.domain.com  ` , or `  domain:example.domain.com  ` .
  - `  PROJECT_ID  ` : The ID of the project.
  - `  KEY_NAME  ` : The display (short) name of your tag key. For example, `  env  ` .
  - `  VALUE_NAME  ` : The display (short) name of your tag value. For example, `  prod  ` .

This command adds an IAM policy binding to the IAM policy of an organization. A policy binding consists of a member, a role, and an optional condition.

For example, to conditionally grant `  user1@example.com  ` the `  spanner.backupAdmin  ` role in all `  123456789012  ` project resources with the tag `  env:prod  ` , run the command:

``` text
gcloud organizations add-iam-policy-binding my-organization
--member=user1@example.com --role=roles/spanner.backupAdmin
--condition=resource.matchTag('123456789012/env', 'prod')
```

### List tags attached to an instance

You can view a list of tag bindings directly attached to or inherited by the instance.

### Console

1.  In the Google Cloud console, go to the **Spanner Instances** page.

2.  Under the list of instances, look for the **Tags** column. Tags are shown in the `  key:value  ` format.

### gcloud

To get a list of tag bindings directly attached to a resource, use the `  gcloud resource-manager tags bindings list  ` command. If you add the `  --effective  ` flag, you also get all the tag bindings inherited by this resource.

To list all tag bindings attached to an instance, run the following command:

``` text
gcloud resource-manager tags bindings list
--parent=//spanner.googleapis.com/projects/PROJECT_ID/instances/INSTANCE_ID
--location=LOCATION
--effective
```

  - `  PROJECT_ID  ` : The ID of the project.
  - `  INSTANCE_ID  ` : The ID of the instance.
  - `  LOCATION  ` : The location of an instance varies depending on its configuration:
      - For a regional instance, the location of its configuration. For example, `  us-east1  ` .
      - For a dual-region instance, the 2-letter resource location. For example, `  de  ` for `  dual-region-germany1  ` .
      - For a multi-region instance, the name of its configuration. For example, `  eur3  ` or `  nam-eur-asia1  ` .

### API

To view a list of tag bindings directly attached to or inherited by the instance using REST or RPC API, see [Listing all tags attached to a resource](/resource-manager/docs/tags/tags-creating-and-managing#api_9) .

### Delete a tag binding

When removing a tag key or value definition, ensure the tag is detached from the instance. You must delete existing tag bindings before deleting the tag.

### Console

1.  In the Google Cloud console, go to the **Spanner Instances** page.

2.  Select the checkbox next to the instance for which you would like to delete a tag binding.

3.  Click label\_important **Tags** .

4.  In the Tags panel, next to the tag you want to detach, click delete **Delete item** .

5.  Click **Save** .

6.  In the **Confirm** dialog, click **Confirm** to detach the tag.
    
    A notification confirms that your tags updated.

### gcloud

To delete a tag binding, run the following command:

``` text
gcloud resource-manager tags bindings delete
--parent=//spanner.googleapis.com/projects/PROJECT_ID/instances/INSTANCE_ID
--tag-value=TAG_VALUE_NAME
--location=LOCATION
```

  - `  PROJECT_ID  ` : The ID of the project.
  - `  INSTANCE_ID  ` : The ID of the instance.
  - `  TAG_VALUE_NAME  ` : `  TAG_VALUE_NAME  ` is the permanent ID or namespace ID of the tag value to be attached. For example: `  tagValues/4567890123  ` (permanent ID) or `  12345678/env/prod  ` (namespace ID). The namespace ID consists of the following:
      - `  ORG_ID  ` : The ID of the organization.
      - `  KEY_NAME  ` : The display (short) name of your tag key. For example, `  env  ` .
      - `  VALUE_NAME  ` : The display (short) name of your tag value. For example, `  prod  ` .
  - `  LOCATION  ` : The location of an instance varies depending on its configuration:
      - For a regional instance, the location of its configuration. For example, `  us-east1  ` .
      - For a dual-region instance, the 2-letter resource location. For example, `  de  ` for `  dual-region-germany1  ` .
      - For a multi-region instance, the name of its configuration. For example, `  eur3  ` or `  nam-eur-asia1  ` .

### API

To detach a tag from a resource by deleting the tag binding resource using REST or RPC API, see [Detaching a tag from a resource](/resource-manager/docs/tags/tags-creating-and-managing#api_10) .

### Delete a tag

After you have deleted your tag binding, you can delete your tag. To delete tag keys and tag values, see [Deleting tags](/resource-manager/docs/tags/tags-creating-and-managing#deleting) .

## What's next

  - Learn more about Google Cloud [tags](/resource-manager/docs/tags/tags-overview) .

  - Learn more about how to [create and manage tags using Resource Manager](/resource-manager/docs/tags/tags-creating-and-managing) .

  - Learn more about [labels](/resource-manager/docs/creating-managing-labels#what-are-labels) , another way to organize your Google Cloud resources.

  - Learn more about [creating IAM allow policies with conditions](/iam/docs/conditions-overview) .
