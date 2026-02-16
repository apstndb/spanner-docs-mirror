## Index

  - `  InstanceAdmin  ` (interface)
  - `  AutoscalingConfig  ` (message)
  - `  AutoscalingConfig.AsymmetricAutoscalingOption  ` (message)
  - `  AutoscalingConfig.AsymmetricAutoscalingOption.AutoscalingConfigOverrides  ` (message)
  - `  AutoscalingConfig.AutoscalingLimits  ` (message)
  - `  AutoscalingConfig.AutoscalingTargets  ` (message)
  - `  CreateInstanceConfigMetadata  ` (message)
  - `  CreateInstanceConfigRequest  ` (message)
  - `  CreateInstanceMetadata  ` (message)
  - `  CreateInstancePartitionMetadata  ` (message)
  - `  CreateInstancePartitionRequest  ` (message)
  - `  CreateInstanceRequest  ` (message)
  - `  DeleteInstanceConfigRequest  ` (message)
  - `  DeleteInstancePartitionRequest  ` (message)
  - `  DeleteInstanceRequest  ` (message)
  - `  EncryptionConfig  ` (message)
  - `  FreeInstanceMetadata  ` (message)
  - `  FreeInstanceMetadata.ExpireBehavior  ` (enum)
  - `  FulfillmentPeriod  ` (enum)
  - `  GetInstanceConfigRequest  ` (message)
  - `  GetInstancePartitionRequest  ` (message)
  - `  GetInstanceRequest  ` (message)
  - `  Instance  ` (message)
  - `  Instance.DefaultBackupScheduleType  ` (enum)
  - `  Instance.Edition  ` (enum)
  - `  Instance.InstanceType  ` (enum)
  - `  Instance.State  ` (enum)
  - `  InstanceConfig  ` (message)
  - `  InstanceConfig.FreeInstanceAvailability  ` (enum)
  - `  InstanceConfig.QuorumType  ` (enum)
  - `  InstanceConfig.State  ` (enum)
  - `  InstanceConfig.Type  ` (enum)
  - `  InstancePartition  ` (message)
  - `  InstancePartition.State  ` (enum)
  - `  ListInstanceConfigOperationsRequest  ` (message)
  - `  ListInstanceConfigOperationsResponse  ` (message)
  - `  ListInstanceConfigsRequest  ` (message)
  - `  ListInstanceConfigsResponse  ` (message)
  - `  ListInstancePartitionOperationsRequest  ` (message)
  - `  ListInstancePartitionOperationsResponse  ` (message)
  - `  ListInstancePartitionsRequest  ` (message)
  - `  ListInstancePartitionsResponse  ` (message)
  - `  ListInstancesRequest  ` (message)
  - `  ListInstancesResponse  ` (message)
  - `  MoveInstanceMetadata  ` (message)
  - `  MoveInstanceRequest  ` (message)
  - `  MoveInstanceRequest.DatabaseMoveConfig  ` (message)
  - `  MoveInstanceResponse  ` (message)
  - `  OperationProgress  ` (message)
  - `  ReplicaComputeCapacity  ` (message)
  - `  ReplicaInfo  ` (message)
  - `  ReplicaInfo.ReplicaType  ` (enum)
  - `  ReplicaSelection  ` (message)
  - `  UpdateInstanceConfigMetadata  ` (message)
  - `  UpdateInstanceConfigRequest  ` (message)
  - `  UpdateInstanceMetadata  ` (message)
  - `  UpdateInstancePartitionMetadata  ` (message)
  - `  UpdateInstancePartitionRequest  ` (message)
  - `  UpdateInstanceRequest  ` (message)

## InstanceAdmin

Cloud Spanner Instance Admin API

The Cloud Spanner Instance Admin API can be used to create, delete, modify and list instances. Instances are dedicated Cloud Spanner serving and storage resources to be used by Cloud Spanner databases.

Each instance has a "configuration", which dictates where the serving resources for the Cloud Spanner instance are located (e.g., US-central, Europe). Configurations are created by Google based on resource availability.

Cloud Spanner billing is based on the instances that exist and their sizes. After an instance exists, there are no additional per-database or per-operation charges for use of the instance (though there may be additional network bandwidth charges). Instances offer isolation: problems with databases in one instance will not affect other instances. However, within an instance databases can affect each other. For example, if one database in an instance receives a lot of requests and consumes most of the instance resources, fewer resources are available for other databases in that instance, and their performance may suffer.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>CreateInstance</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc CreateInstance(                         CreateInstanceRequest            </code> ) returns ( <code dir="ltr" translate="no">              Operation            </code> )</p>
<p>Creates an instance and begins preparing it to begin serving. The returned long-running operation can be used to track the progress of preparing the new instance. The instance name is assigned by the caller. If the named instance already exists, <code dir="ltr" translate="no">           CreateInstance          </code> returns <code dir="ltr" translate="no">           ALREADY_EXISTS          </code> .</p>
<p>Immediately upon completion of this request:</p>
<ul>
<li>The instance is readable via the API, with all requested attributes but no allocated resources. Its state is <code dir="ltr" translate="no">            CREATING           </code> .</li>
</ul>
<p>Until completion of the returned operation:</p>
<ul>
<li>Cancelling the operation renders the instance immediately unreadable via the API.</li>
<li>The instance can be deleted.</li>
<li>All other attempts to modify the instance are rejected.</li>
</ul>
<p>Upon completion of the returned operation:</p>
<ul>
<li>Billing for all successfully-allocated resources begins (some types may have lower than the requested levels).</li>
<li>Databases can be created in the instance.</li>
<li>The instance's allocated resource levels are readable via the API.</li>
<li>The instance's state becomes <code dir="ltr" translate="no">            READY           </code> .</li>
</ul>
<p>The returned long-running operation will have a name of the format <code dir="ltr" translate="no">           &lt;instance_name&gt;/operations/&lt;operation_id&gt;          </code> and can be used to track creation of the instance. The metadata field type is <code dir="ltr" translate="no">             CreateInstanceMetadata           </code> . The response field type is <code dir="ltr" translate="no">             Instance           </code> , if successful.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>CreateInstanceConfig</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc CreateInstanceConfig(                         CreateInstanceConfigRequest            </code> ) returns ( <code dir="ltr" translate="no">              Operation            </code> )</p>
<p>Creates an instance configuration and begins preparing it to be used. The returned long-running operation can be used to track the progress of preparing the new instance configuration. The instance configuration name is assigned by the caller. If the named instance configuration already exists, <code dir="ltr" translate="no">           CreateInstanceConfig          </code> returns <code dir="ltr" translate="no">           ALREADY_EXISTS          </code> .</p>
<p>Immediately after the request returns:</p>
<ul>
<li>The instance configuration is readable via the API, with all requested attributes. The instance configuration's <code dir="ltr" translate="no">              reconciling            </code> field is set to true. Its state is <code dir="ltr" translate="no">            CREATING           </code> .</li>
</ul>
<p>While the operation is pending:</p>
<ul>
<li>Cancelling the operation renders the instance configuration immediately unreadable via the API.</li>
<li>Except for deleting the creating resource, all other attempts to modify the instance configuration are rejected.</li>
</ul>
<p>Upon completion of the returned operation:</p>
<ul>
<li>Instances can be created using the instance configuration.</li>
<li>The instance configuration's <code dir="ltr" translate="no">              reconciling            </code> field becomes false. Its state becomes <code dir="ltr" translate="no">            READY           </code> .</li>
</ul>
<p>The returned long-running operation will have a name of the format <code dir="ltr" translate="no">           &lt;instance_config_name&gt;/operations/&lt;operation_id&gt;          </code> and can be used to track creation of the instance configuration. The metadata field type is <code dir="ltr" translate="no">             CreateInstanceConfigMetadata           </code> . The response field type is <code dir="ltr" translate="no">             InstanceConfig           </code> , if successful.</p>
<p>Authorization requires <code dir="ltr" translate="no">           spanner.instanceConfigs.create          </code> permission on the resource <code dir="ltr" translate="no">             parent           </code> .</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>CreateInstancePartition</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc CreateInstancePartition(                         CreateInstancePartitionRequest            </code> ) returns ( <code dir="ltr" translate="no">              Operation            </code> )</p>
<p>Creates an instance partition and begins preparing it to be used. The returned long-running operation can be used to track the progress of preparing the new instance partition. The instance partition name is assigned by the caller. If the named instance partition already exists, <code dir="ltr" translate="no">           CreateInstancePartition          </code> returns <code dir="ltr" translate="no">           ALREADY_EXISTS          </code> .</p>
<p>Immediately upon completion of this request:</p>
<ul>
<li>The instance partition is readable via the API, with all requested attributes but no allocated resources. Its state is <code dir="ltr" translate="no">            CREATING           </code> .</li>
</ul>
<p>Until completion of the returned operation:</p>
<ul>
<li>Cancelling the operation renders the instance partition immediately unreadable via the API.</li>
<li>The instance partition can be deleted.</li>
<li>All other attempts to modify the instance partition are rejected.</li>
</ul>
<p>Upon completion of the returned operation:</p>
<ul>
<li>Billing for all successfully-allocated resources begins (some types may have lower than the requested levels).</li>
<li>Databases can start using this instance partition.</li>
<li>The instance partition's allocated resource levels are readable via the API.</li>
<li>The instance partition's state becomes <code dir="ltr" translate="no">            READY           </code> .</li>
</ul>
<p>The returned long-running operation will have a name of the format <code dir="ltr" translate="no">           &lt;instance_partition_name&gt;/operations/&lt;operation_id&gt;          </code> and can be used to track creation of the instance partition. The metadata field type is <code dir="ltr" translate="no">             CreateInstancePartitionMetadata           </code> . The response field type is <code dir="ltr" translate="no">             InstancePartition           </code> , if successful.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>DeleteInstance</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc DeleteInstance(                         DeleteInstanceRequest            </code> ) returns ( <code dir="ltr" translate="no">              Empty            </code> )</p>
<p>Deletes an instance.</p>
<p>Immediately upon completion of the request:</p>
<ul>
<li>Billing ceases for all of the instance's reserved resources.</li>
</ul>
<p>Soon afterward:</p>
<ul>
<li>The instance and <em>all of its databases</em> immediately and irrevocably disappear from the API. All data in the databases is permanently deleted.</li>
</ul>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>DeleteInstanceConfig</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc DeleteInstanceConfig(                         DeleteInstanceConfigRequest            </code> ) returns ( <code dir="ltr" translate="no">              Empty            </code> )</p>
<p>Deletes the instance configuration. Deletion is only allowed when no instances are using the configuration. If any instances are using the configuration, returns <code dir="ltr" translate="no">           FAILED_PRECONDITION          </code> .</p>
<p>Only user-managed configurations can be deleted.</p>
<p>Authorization requires <code dir="ltr" translate="no">           spanner.instanceConfigs.delete          </code> permission on the resource <code dir="ltr" translate="no">             name           </code> .</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>DeleteInstancePartition</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc DeleteInstancePartition(                         DeleteInstancePartitionRequest            </code> ) returns ( <code dir="ltr" translate="no">              Empty            </code> )</p>
<p>Deletes an existing instance partition. Requires that the instance partition is not used by any database or backup and is not the default instance partition of an instance.</p>
<p>Authorization requires <code dir="ltr" translate="no">           spanner.instancePartitions.delete          </code> permission on the resource <code dir="ltr" translate="no">             name           </code> .</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>GetIamPolicy</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc GetIamPolicy(                         GetIamPolicyRequest            </code> ) returns ( <code dir="ltr" translate="no">              Policy            </code> )</p>
<p>Gets the access control policy for an instance resource. Returns an empty policy if an instance exists but does not have a policy set.</p>
<p>Authorization requires <code dir="ltr" translate="no">           spanner.instances.getIamPolicy          </code> on <code dir="ltr" translate="no">             resource           </code> .</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>GetInstance</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc GetInstance(                         GetInstanceRequest            </code> ) returns ( <code dir="ltr" translate="no">              Instance            </code> )</p>
<p>Gets information about a particular instance.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>GetInstanceConfig</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc GetInstanceConfig(                         GetInstanceConfigRequest            </code> ) returns ( <code dir="ltr" translate="no">              InstanceConfig            </code> )</p>
<p>Gets information about a particular instance configuration.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>GetInstancePartition</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc GetInstancePartition(                         GetInstancePartitionRequest            </code> ) returns ( <code dir="ltr" translate="no">              InstancePartition            </code> )</p>
<p>Gets information about a particular instance partition.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>ListInstanceConfigOperations</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc ListInstanceConfigOperations(                         ListInstanceConfigOperationsRequest            </code> ) returns ( <code dir="ltr" translate="no">              ListInstanceConfigOperationsResponse            </code> )</p>
<p>Lists the user-managed instance configuration long-running operations in the given project. An instance configuration operation has a name of the form <code dir="ltr" translate="no">           projects/&lt;project&gt;/instanceConfigs/&lt;instance_config&gt;/operations/&lt;operation&gt;          </code> . The long-running operation metadata field type <code dir="ltr" translate="no">           metadata.type_url          </code> describes the type of the metadata. Operations returned include those that have completed/failed/canceled within the last 7 days, and pending operations. Operations returned are ordered by <code dir="ltr" translate="no">           operation.metadata.value.start_time          </code> in descending order starting from the most recently started operation.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>ListInstanceConfigs</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc ListInstanceConfigs(                         ListInstanceConfigsRequest            </code> ) returns ( <code dir="ltr" translate="no">              ListInstanceConfigsResponse            </code> )</p>
<p>Lists the supported instance configurations for a given project.</p>
<p>Returns both Google-managed configurations and user-managed configurations.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>ListInstancePartitionOperations</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc ListInstancePartitionOperations(                         ListInstancePartitionOperationsRequest            </code> ) returns ( <code dir="ltr" translate="no">              ListInstancePartitionOperationsResponse            </code> )</p>
<p>Lists instance partition long-running operations in the given instance. An instance partition operation has a name of the form <code dir="ltr" translate="no">           projects/&lt;project&gt;/instances/&lt;instance&gt;/instancePartitions/&lt;instance_partition&gt;/operations/&lt;operation&gt;          </code> . The long-running operation metadata field type <code dir="ltr" translate="no">           metadata.type_url          </code> describes the type of the metadata. Operations returned include those that have completed/failed/canceled within the last 7 days, and pending operations. Operations returned are ordered by <code dir="ltr" translate="no">           operation.metadata.value.start_time          </code> in descending order starting from the most recently started operation.</p>
<p>Authorization requires <code dir="ltr" translate="no">           spanner.instancePartitionOperations.list          </code> permission on the resource <code dir="ltr" translate="no">             parent           </code> .</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>ListInstancePartitions</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc ListInstancePartitions(                         ListInstancePartitionsRequest            </code> ) returns ( <code dir="ltr" translate="no">              ListInstancePartitionsResponse            </code> )</p>
<p>Lists all instance partitions for the given instance.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>ListInstances</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc ListInstances(                         ListInstancesRequest            </code> ) returns ( <code dir="ltr" translate="no">              ListInstancesResponse            </code> )</p>
<p>Lists all instances in the given project.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>MoveInstance</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc MoveInstance(                         MoveInstanceRequest            </code> ) returns ( <code dir="ltr" translate="no">              Operation            </code> )</p>
<p>Moves an instance to the target instance configuration. You can use the returned long-running operation to track the progress of moving the instance.</p>
<p><code dir="ltr" translate="no">           MoveInstance          </code> returns <code dir="ltr" translate="no">           FAILED_PRECONDITION          </code> if the instance meets any of the following criteria:</p>
<ul>
<li>Is undergoing a move to a different instance configuration</li>
<li>Has backups</li>
<li>Has an ongoing update</li>
<li>Contains any CMEK-enabled databases</li>
<li>Is a free trial instance</li>
</ul>
<p>While the operation is pending:</p>
<ul>
<li>All other attempts to modify the instance, including changes to its compute capacity, are rejected.</li>
<li>The following database and backup admin operations are rejected:</li>
</ul>
<pre class="text" dir="ltr" data-is-upgraded="" translate="no"><code>* `DatabaseAdmin.CreateDatabase`
* `DatabaseAdmin.UpdateDatabaseDdl` (disabled if default_leader is
   specified in the request.)
* `DatabaseAdmin.RestoreDatabase`
* `DatabaseAdmin.CreateBackup`
* `DatabaseAdmin.CopyBackup`</code></pre>
<ul>
<li>Both the source and target instance configurations are subject to hourly compute and storage charges.</li>
<li>The instance might experience higher read-write latencies and a higher transaction abort rate. However, moving an instance doesn't cause any downtime.</li>
</ul>
<p>The returned long-running operation has a name of the format <code dir="ltr" translate="no">           &lt;instance_name&gt;/operations/&lt;operation_id&gt;          </code> and can be used to track the move instance operation. The metadata field type is <code dir="ltr" translate="no">             MoveInstanceMetadata           </code> . The response field type is <code dir="ltr" translate="no">             Instance           </code> , if successful. Cancelling the operation sets its metadata's <code dir="ltr" translate="no">             cancel_time           </code> . Cancellation is not immediate because it involves moving any data previously moved to the target instance configuration back to the original instance configuration. You can use this operation to track the progress of the cancellation. Upon successful completion of the cancellation, the operation terminates with <code dir="ltr" translate="no">           CANCELLED          </code> status.</p>
<p>If not cancelled, upon completion of the returned operation:</p>
<ul>
<li>The instance successfully moves to the target instance configuration.</li>
<li>You are billed for compute and storage in target instance configuration.</li>
</ul>
<p>Authorization requires the <code dir="ltr" translate="no">           spanner.instances.update          </code> permission on the resource <code dir="ltr" translate="no">             instance           </code> .</p>
<p>For more details, see <a href="https://cloud.google.com/spanner/docs/move-instance">Move an instance</a> .</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>SetIamPolicy</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc SetIamPolicy(                         SetIamPolicyRequest            </code> ) returns ( <code dir="ltr" translate="no">              Policy            </code> )</p>
<p>Sets the access control policy on an instance resource. Replaces any existing policy.</p>
<p>Authorization requires <code dir="ltr" translate="no">           spanner.instances.setIamPolicy          </code> on <code dir="ltr" translate="no">             resource           </code> .</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>TestIamPermissions</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc TestIamPermissions(                         TestIamPermissionsRequest            </code> ) returns ( <code dir="ltr" translate="no">              TestIamPermissionsResponse            </code> )</p>
<p>Returns permissions that the caller has on the specified instance resource.</p>
<p>Attempting this RPC on a non-existent Cloud Spanner instance resource will result in a NOT_FOUND error if the user has <code dir="ltr" translate="no">           spanner.instances.list          </code> permission on the containing Google Cloud Project. Otherwise returns an empty set of permissions.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>UpdateInstance</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc UpdateInstance(                         UpdateInstanceRequest            </code> ) returns ( <code dir="ltr" translate="no">              Operation            </code> )</p>
<p>Updates an instance, and begins allocating or releasing resources as requested. The returned long-running operation can be used to track the progress of updating the instance. If the named instance does not exist, returns <code dir="ltr" translate="no">           NOT_FOUND          </code> .</p>
<p>Immediately upon completion of this request:</p>
<ul>
<li>For resource types for which a decrease in the instance's allocation has been requested, billing is based on the newly-requested level.</li>
</ul>
<p>Until completion of the returned operation:</p>
<ul>
<li>Cancelling the operation sets its metadata's <code dir="ltr" translate="no">              cancel_time            </code> , and begins restoring resources to their pre-request values. The operation is guaranteed to succeed at undoing all resource changes, after which point it terminates with a <code dir="ltr" translate="no">            CANCELLED           </code> status.</li>
<li>All other attempts to modify the instance are rejected.</li>
<li>Reading the instance via the API continues to give the pre-request resource levels.</li>
</ul>
<p>Upon completion of the returned operation:</p>
<ul>
<li>Billing begins for all successfully-allocated resources (some types may have lower than the requested levels).</li>
<li>All newly-reserved resources are available for serving the instance's tables.</li>
<li>The instance's new resource levels are readable via the API.</li>
</ul>
<p>The returned long-running operation will have a name of the format <code dir="ltr" translate="no">           &lt;instance_name&gt;/operations/&lt;operation_id&gt;          </code> and can be used to track the instance modification. The metadata field type is <code dir="ltr" translate="no">             UpdateInstanceMetadata           </code> . The response field type is <code dir="ltr" translate="no">             Instance           </code> , if successful.</p>
<p>Authorization requires <code dir="ltr" translate="no">           spanner.instances.update          </code> permission on the resource <code dir="ltr" translate="no">             name           </code> .</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>UpdateInstanceConfig</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc UpdateInstanceConfig(                         UpdateInstanceConfigRequest            </code> ) returns ( <code dir="ltr" translate="no">              Operation            </code> )</p>
<p>Updates an instance configuration. The returned long-running operation can be used to track the progress of updating the instance. If the named instance configuration does not exist, returns <code dir="ltr" translate="no">           NOT_FOUND          </code> .</p>
<p>Only user-managed configurations can be updated.</p>
<p>Immediately after the request returns:</p>
<ul>
<li>The instance configuration's <code dir="ltr" translate="no">              reconciling            </code> field is set to true.</li>
</ul>
<p>While the operation is pending:</p>
<ul>
<li>Cancelling the operation sets its metadata's <code dir="ltr" translate="no">              cancel_time            </code> . The operation is guaranteed to succeed at undoing all changes, after which point it terminates with a <code dir="ltr" translate="no">            CANCELLED           </code> status.</li>
<li>All other attempts to modify the instance configuration are rejected.</li>
<li>Reading the instance configuration via the API continues to give the pre-request values.</li>
</ul>
<p>Upon completion of the returned operation:</p>
<ul>
<li>Creating instances using the instance configuration uses the new values.</li>
<li>The new values of the instance configuration are readable via the API.</li>
<li>The instance configuration's <code dir="ltr" translate="no">              reconciling            </code> field becomes false.</li>
</ul>
<p>The returned long-running operation will have a name of the format <code dir="ltr" translate="no">           &lt;instance_config_name&gt;/operations/&lt;operation_id&gt;          </code> and can be used to track the instance configuration modification. The metadata field type is <code dir="ltr" translate="no">             UpdateInstanceConfigMetadata           </code> . The response field type is <code dir="ltr" translate="no">             InstanceConfig           </code> , if successful.</p>
<p>Authorization requires <code dir="ltr" translate="no">           spanner.instanceConfigs.update          </code> permission on the resource <code dir="ltr" translate="no">             name           </code> .</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>UpdateInstancePartition</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc UpdateInstancePartition(                         UpdateInstancePartitionRequest            </code> ) returns ( <code dir="ltr" translate="no">              Operation            </code> )</p>
<p>Updates an instance partition, and begins allocating or releasing resources as requested. The returned long-running operation can be used to track the progress of updating the instance partition. If the named instance partition does not exist, returns <code dir="ltr" translate="no">           NOT_FOUND          </code> .</p>
<p>Immediately upon completion of this request:</p>
<ul>
<li>For resource types for which a decrease in the instance partition's allocation has been requested, billing is based on the newly-requested level.</li>
</ul>
<p>Until completion of the returned operation:</p>
<ul>
<li>Cancelling the operation sets its metadata's <code dir="ltr" translate="no">              cancel_time            </code> , and begins restoring resources to their pre-request values. The operation is guaranteed to succeed at undoing all resource changes, after which point it terminates with a <code dir="ltr" translate="no">            CANCELLED           </code> status.</li>
<li>All other attempts to modify the instance partition are rejected.</li>
<li>Reading the instance partition via the API continues to give the pre-request resource levels.</li>
</ul>
<p>Upon completion of the returned operation:</p>
<ul>
<li>Billing begins for all successfully-allocated resources (some types may have lower than the requested levels).</li>
<li>All newly-reserved resources are available for serving the instance partition's tables.</li>
<li>The instance partition's new resource levels are readable via the API.</li>
</ul>
<p>The returned long-running operation will have a name of the format <code dir="ltr" translate="no">           &lt;instance_partition_name&gt;/operations/&lt;operation_id&gt;          </code> and can be used to track the instance partition modification. The metadata field type is <code dir="ltr" translate="no">             UpdateInstancePartitionMetadata           </code> . The response field type is <code dir="ltr" translate="no">             InstancePartition           </code> , if successful.</p>
<p>Authorization requires <code dir="ltr" translate="no">           spanner.instancePartitions.update          </code> permission on the resource <code dir="ltr" translate="no">             name           </code> .</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

## AutoscalingConfig

Autoscaling configuration for an instance.

Fields

`  autoscaling_limits  `

`  AutoscalingLimits  `

Required. Autoscaling limits for an instance.

`  autoscaling_targets  `

`  AutoscalingTargets  `

Required. The autoscaling targets for an instance.

`  asymmetric_autoscaling_options[]  `

`  AsymmetricAutoscalingOption  `

Optional. Optional asymmetric autoscaling options. Replicas matching the replica selection criteria will be autoscaled independently from other replicas. The autoscaler will scale the replicas based on the utilization of replicas identified by the replica selection. Replica selections should not overlap with each other.

Other replicas (those do not match any replica selection) will be autoscaled together and will have the same compute capacity allocated to them.

## AsymmetricAutoscalingOption

AsymmetricAutoscalingOption specifies the scaling of replicas identified by the given selection.

Fields

`  replica_selection  `

`  ReplicaSelection  `

Required. Selects the replicas to which this AsymmetricAutoscalingOption applies. Only read-only replicas are supported.

`  overrides  `

`  AutoscalingConfigOverrides  `

Optional. Overrides applied to the top-level autoscaling configuration for the selected replicas.

## AutoscalingConfigOverrides

Overrides the top-level autoscaling configuration for the replicas identified by `  replica_selection  ` . All fields in this message are optional. Any unspecified fields will use the corresponding values from the top-level autoscaling configuration.

Fields

`  autoscaling_limits  `

`  AutoscalingLimits  `

Optional. If specified, overrides the min/max limit in the top-level autoscaling configuration for the selected replicas.

`  autoscaling_target_high_priority_cpu_utilization_percent  `

`  int32  `

Optional. If specified, overrides the autoscaling target high\_priority\_cpu\_utilization\_percent in the top-level autoscaling configuration for the selected replicas.

## AutoscalingLimits

The autoscaling limits for the instance. Users can define the minimum and maximum compute capacity allocated to the instance, and the autoscaler will only scale within that range. Users can either use nodes or processing units to specify the limits, but should use the same unit to set both the min\_limit and max\_limit.

Fields

Union field `  min_limit  ` . The minimum compute capacity for the instance. `  min_limit  ` can be only one of the following:

`  min_nodes  `

`  int32  `

Minimum number of nodes allocated to the instance. If set, this number should be greater than or equal to 1.

`  min_processing_units  `

`  int32  `

Minimum number of processing units allocated to the instance. If set, this number should be multiples of 1000.

Union field `  max_limit  ` . The maximum compute capacity for the instance. The maximum compute capacity should be less than or equal to 10X the minimum compute capacity. `  max_limit  ` can be only one of the following:

`  max_nodes  `

`  int32  `

Maximum number of nodes allocated to the instance. If set, this number should be greater than or equal to min\_nodes.

`  max_processing_units  `

`  int32  `

Maximum number of processing units allocated to the instance. If set, this number should be multiples of 1000 and be greater than or equal to min\_processing\_units.

## AutoscalingTargets

The autoscaling targets for an instance.

Fields

`  high_priority_cpu_utilization_percent  `

`  int32  `

Required. The target high priority cpu utilization percentage that the autoscaler should be trying to achieve for the instance. This number is on a scale from 0 (no utilization) to 100 (full utilization). The valid range is \[10, 90\] inclusive.

`  storage_utilization_percent  `

`  int32  `

Required. The target storage utilization percentage that the autoscaler should be trying to achieve for the instance. This number is on a scale from 0 (no utilization) to 100 (full utilization). The valid range is \[10, 99\] inclusive.

## CreateInstanceConfigMetadata

Metadata type for the operation returned by `  CreateInstanceConfig  ` .

Fields

`  instance_config  `

`  InstanceConfig  `

The target instance configuration end state.

`  progress  `

`  OperationProgress  `

The progress of the `  CreateInstanceConfig  ` operation.

`  cancel_time  `

`  Timestamp  `

The time at which this operation was cancelled.

## CreateInstanceConfigRequest

The request for `  CreateInstanceConfig  ` .

Fields

`  parent  `

`  string  `

Required. The name of the project in which to create the instance configuration. Values are of the form `  projects/<project>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  parent  ` :

  - `  spanner.instanceConfigs.create  `

`  instance_config_id  `

`  string  `

Required. The ID of the instance configuration to create. Valid identifiers are of the form `  custom-[-a-z0-9]*[a-z0-9]  ` and must be between 2 and 64 characters in length. The `  custom-  ` prefix is required to avoid name conflicts with Google-managed configurations.

`  instance_config  `

`  InstanceConfig  `

Required. The `  InstanceConfig  ` proto of the configuration to create. `  instance_config.name  ` must be `  <parent>/instanceConfigs/<instance_config_id>  ` . `  instance_config.base_config  ` must be a Google-managed configuration name, e.g. /instanceConfigs/us-east1, /instanceConfigs/nam3.

`  validate_only  `

`  bool  `

An option to validate, but not actually execute, a request, and provide the same response.

## CreateInstanceMetadata

Metadata type for the operation returned by `  CreateInstance  ` .

Fields

`  instance  `

`  Instance  `

The instance being created.

`  start_time  `

`  Timestamp  `

The time at which the `  CreateInstance  ` request was received.

`  cancel_time  `

`  Timestamp  `

The time at which this operation was cancelled. If set, this operation is in the process of undoing itself (which is guaranteed to succeed) and cannot be cancelled again.

`  end_time  `

`  Timestamp  `

The time at which this operation failed or was completed successfully.

`  expected_fulfillment_period  `

`  FulfillmentPeriod  `

The expected fulfillment period of this create operation.

## CreateInstancePartitionMetadata

Metadata type for the operation returned by `  CreateInstancePartition  ` .

Fields

`  instance_partition  `

`  InstancePartition  `

The instance partition being created.

`  start_time  `

`  Timestamp  `

The time at which the `  CreateInstancePartition  ` request was received.

`  cancel_time  `

`  Timestamp  `

The time at which this operation was cancelled. If set, this operation is in the process of undoing itself (which is guaranteed to succeed) and cannot be cancelled again.

`  end_time  `

`  Timestamp  `

The time at which this operation failed or was completed successfully.

## CreateInstancePartitionRequest

The request for `  CreateInstancePartition  ` .

Fields

`  parent  `

`  string  `

Required. The name of the instance in which to create the instance partition. Values are of the form `  projects/<project>/instances/<instance>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  parent  ` :

  - `  spanner.instancePartitions.create  `

`  instance_partition_id  `

`  string  `

Required. The ID of the instance partition to create. Valid identifiers are of the form `  [a-z][-a-z0-9]*[a-z0-9]  ` and must be between 2 and 64 characters in length.

`  instance_partition  `

`  InstancePartition  `

Required. The instance partition to create. The instance\_partition.name may be omitted, but if specified must be `  <parent>/instancePartitions/<instance_partition_id>  ` .

## CreateInstanceRequest

The request for `  CreateInstance  ` .

Fields

`  parent  `

`  string  `

Required. The name of the project in which to create the instance. Values are of the form `  projects/<project>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  parent  ` :

  - `  spanner.instances.create  `

`  instance_id  `

`  string  `

Required. The ID of the instance to create. Valid identifiers are of the form `  [a-z][-a-z0-9]*[a-z0-9]  ` and must be between 2 and 64 characters in length.

`  instance  `

`  Instance  `

Required. The instance to create. The name may be omitted, but if specified must be `  <parent>/instances/<instance_id>  ` .

## DeleteInstanceConfigRequest

The request for `  DeleteInstanceConfig  ` .

Fields

`  name  `

`  string  `

Required. The name of the instance configuration to be deleted. Values are of the form `  projects/<project>/instanceConfigs/<instance_config>  `

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  name  ` :

  - `  spanner.instanceConfigs.delete  `

`  etag  `

`  string  `

Used for optimistic concurrency control as a way to help prevent simultaneous deletes of an instance configuration from overwriting each other. If not empty, the API only deletes the instance configuration when the etag provided matches the current status of the requested instance configuration. Otherwise, deletes the instance configuration without checking the current status of the requested instance configuration.

`  validate_only  `

`  bool  `

An option to validate, but not actually execute, a request, and provide the same response.

## DeleteInstancePartitionRequest

The request for `  DeleteInstancePartition  ` .

Fields

`  name  `

`  string  `

Required. The name of the instance partition to be deleted. Values are of the form `  projects/{project}/instances/{instance}/instancePartitions/{instance_partition}  `

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  name  ` :

  - `  spanner.instancePartitions.delete  `

`  etag  `

`  string  `

Optional. If not empty, the API only deletes the instance partition when the etag provided matches the current status of the requested instance partition. Otherwise, deletes the instance partition without checking the current status of the requested instance partition.

## DeleteInstanceRequest

The request for `  DeleteInstance  ` .

Fields

`  name  `

`  string  `

Required. The name of the instance to be deleted. Values are of the form `  projects/<project>/instances/<instance>  `

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  name  ` :

  - `  spanner.instances.delete  `

## EncryptionConfig

Encryption configuration for a Cloud Spanner database.

Fields

`  kms_key_name  `

`  string  `

Optional. This field is maintained for backwards compatibility. For new callers, we recommend using `  kms_key_names  ` to specify the KMS key. Only use `  kms_key_name  ` if the location of the KMS key matches the database instance's configuration (location) exactly. For example, if the KMS location is in `  us-central1  ` or `  nam3  ` , then the database instance must also be in `  us-central1  ` or `  nam3  ` .

The Cloud KMS key that is used to encrypt and decrypt the restored database. Values are of the form `  projects/<project>/locations/<location>/keyRings/<key_ring>/cryptoKeys/<kms_key_name>  ` .

`  kms_key_names[]  `

`  string  `

Optional. Specifies the KMS configuration for one or more keys used to encrypt the database. Values are of the form `  projects/<project>/locations/<location>/keyRings/<key_ring>/cryptoKeys/<kms_key_name>  ` .

The keys referenced by `  kms_key_names  ` must fully cover all regions of the database's instance configuration. Some examples:

  - For regional (single-region) instance configurations, specify a regional location KMS key.
  - For multi-region instance configurations of type `  GOOGLE_MANAGED  ` , either specify a multi-region location KMS key or multiple regional location KMS keys that cover all regions in the instance configuration.
  - For an instance configuration of type `  USER_MANAGED  ` , specify only regional location KMS keys to cover each region in the instance configuration. Multi-region location KMS keys aren't supported for `  USER_MANAGED  ` type instance configurations.

## FreeInstanceMetadata

Free instance specific metadata that is kept even after an instance has been upgraded for tracking purposes.

Fields

`  expire_time  `

`  Timestamp  `

Output only. Timestamp after which the instance will either be upgraded or scheduled for deletion after a grace period. ExpireBehavior is used to choose between upgrading or scheduling the free instance for deletion. This timestamp is set during the creation of a free instance.

`  upgrade_time  `

`  Timestamp  `

Output only. If present, the timestamp at which the free instance was upgraded to a provisioned instance.

`  expire_behavior  `

`  ExpireBehavior  `

Specifies the expiration behavior of a free instance. The default of ExpireBehavior is `  REMOVE_AFTER_GRACE_PERIOD  ` . This can be modified during or after creation, and before expiration.

## ExpireBehavior

Allows users to change behavior when a free instance expires.

Enums

`  EXPIRE_BEHAVIOR_UNSPECIFIED  `

Not specified.

`  FREE_TO_PROVISIONED  `

When the free instance expires, upgrade the instance to a provisioned instance.

`  REMOVE_AFTER_GRACE_PERIOD  `

When the free instance expires, disable the instance, and delete it after the grace period passes if it has not been upgraded.

## FulfillmentPeriod

Indicates the expected fulfillment period of an operation.

Enums

`  FULFILLMENT_PERIOD_UNSPECIFIED  `

Not specified.

`  FULFILLMENT_PERIOD_NORMAL  `

Normal fulfillment period. The operation is expected to complete within minutes.

`  FULFILLMENT_PERIOD_EXTENDED  `

Extended fulfillment period. It can take up to an hour for the operation to complete.

## GetInstanceConfigRequest

The request for `  GetInstanceConfigRequest  ` .

Fields

`  name  `

`  string  `

Required. The name of the requested instance configuration. Values are of the form `  projects/<project>/instanceConfigs/<config>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  name  ` :

  - `  spanner.instanceConfigs.get  `

## GetInstancePartitionRequest

The request for `  GetInstancePartition  ` .

Fields

`  name  `

`  string  `

Required. The name of the requested instance partition. Values are of the form `  projects/{project}/instances/{instance}/instancePartitions/{instance_partition}  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  name  ` :

  - `  spanner.instancePartitions.get  `

## GetInstanceRequest

The request for `  GetInstance  ` .

Fields

`  name  `

`  string  `

Required. The name of the requested instance. Values are of the form `  projects/<project>/instances/<instance>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  name  ` :

  - `  spanner.instances.get  `

`  field_mask  `

`  FieldMask  `

If field\_mask is present, specifies the subset of `  Instance  ` fields that should be returned. If absent, all `  Instance  ` fields are returned.

## Instance

An isolated set of Cloud Spanner resources on which databases can be hosted.

Fields

`  name  `

`  string  `

Required. A unique identifier for the instance, which cannot be changed after the instance is created. Values are of the form `  projects/<project>/instances/[a-z][-a-z0-9]*[a-z0-9]  ` . The final segment of the name must be between 2 and 64 characters in length.

`  config  `

`  string  `

Required. The name of the instance's configuration. Values are of the form `  projects/<project>/instanceConfigs/<configuration>  ` . See also `  InstanceConfig  ` and `  ListInstanceConfigs  ` .

`  display_name  `

`  string  `

Required. The descriptive name for this instance as it appears in UIs. Must be unique per project and between 4 and 30 characters in length.

`  node_count  `

`  int32  `

The number of nodes allocated to this instance. At most, one of either `  node_count  ` or `  processing_units  ` should be present in the message.

Users can set the `  node_count  ` field to specify the target number of nodes allocated to the instance.

If autoscaling is enabled, `  node_count  ` is treated as an `  OUTPUT_ONLY  ` field and reflects the current number of nodes allocated to the instance.

This might be zero in API responses for instances that are not yet in the `  READY  ` state.

If the instance has varying node count across replicas (achieved by setting `  asymmetric_autoscaling_options  ` in the autoscaling configuration), the `  node_count  ` set here is the maximum node count across all replicas.

For more information, see [Compute capacity, nodes, and processing units](https://cloud.google.com/spanner/docs/compute-capacity) .

`  processing_units  `

`  int32  `

The number of processing units allocated to this instance. At most, one of either `  processing_units  ` or `  node_count  ` should be present in the message.

Users can set the `  processing_units  ` field to specify the target number of processing units allocated to the instance.

If autoscaling is enabled, `  processing_units  ` is treated as an `  OUTPUT_ONLY  ` field and reflects the current number of processing units allocated to the instance.

This might be zero in API responses for instances that are not yet in the `  READY  ` state.

If the instance has varying processing units per replica (achieved by setting `  asymmetric_autoscaling_options  ` in the autoscaling configuration), the `  processing_units  ` set here is the maximum processing units across all replicas.

For more information, see [Compute capacity, nodes and processing units](https://cloud.google.com/spanner/docs/compute-capacity) .

`  replica_compute_capacity[]  `

`  ReplicaComputeCapacity  `

Output only. Lists the compute capacity per ReplicaSelection. A replica selection identifies a set of replicas with common properties. Replicas identified by a ReplicaSelection are scaled with the same compute capacity.

`  autoscaling_config  `

`  AutoscalingConfig  `

Optional. The autoscaling configuration. Autoscaling is enabled if this field is set. When autoscaling is enabled, node\_count and processing\_units are treated as OUTPUT\_ONLY fields and reflect the current compute capacity allocated to the instance.

`  state  `

`  State  `

Output only. The current instance state. For `  CreateInstance  ` , the state must be either omitted or set to `  CREATING  ` . For `  UpdateInstance  ` , the state must be either omitted or set to `  READY  ` .

`  labels  `

`  map<string, string>  `

Cloud Labels are a flexible and lightweight mechanism for organizing cloud resources into groups that reflect a customer's organizational needs and deployment strategies. Cloud Labels can be used to filter collections of resources. They can be used to control how resource metrics are aggregated. And they can be used as arguments to policy management rules (e.g. route, firewall, load balancing, etc.).

  - Label keys must be between 1 and 63 characters long and must conform to the following regular expression: `  [a-z][a-z0-9_-]{0,62}  ` .
  - Label values must be between 0 and 63 characters long and must conform to the regular expression `  [a-z0-9_-]{0,63}  ` .
  - No more than 64 labels can be associated with a given resource.

See <https://goo.gl/xmQnxf> for more information on and examples of labels.

If you plan to use labels in your own code, please note that additional characters may be allowed in the future. And so you are advised to use an internal label representation, such as JSON, which doesn't rely upon specific characters being disallowed. For example, representing labels as the string: name + "\_" + value would prove problematic if we were to allow "\_" in a future release.

`  instance_type  `

`  InstanceType  `

The `  InstanceType  ` of the current instance.

`  endpoint_uris[]  `

`  string  `

Deprecated. This field is not populated.

`  create_time  `

`  Timestamp  `

Output only. The time at which the instance was created.

`  update_time  `

`  Timestamp  `

Output only. The time at which the instance was most recently updated.

`  free_instance_metadata  `

`  FreeInstanceMetadata  `

Free instance metadata. Only populated for free instances.

`  edition  `

`  Edition  `

Optional. The `  Edition  ` of the current instance.

`  default_backup_schedule_type  `

`  DefaultBackupScheduleType  `

Optional. Controls the default backup schedule behavior for new databases within the instance. By default, a backup schedule is created automatically when a new database is created in a new instance.

Note that the `  AUTOMATIC  ` value isn't permitted for free instances, as backups and backup schedules aren't supported for free instances.

In the `  GetInstance  ` or `  ListInstances  ` response, if the value of `  default_backup_schedule_type  ` isn't set, or set to `  NONE  ` , Spanner doesn't create a default backup schedule for new databases in the instance.

## DefaultBackupScheduleType

Indicates the [default backup schedule](https://cloud.google.com/spanner/docs/backup#default-backup-schedules) behavior for new databases within the instance.

Enums

`  DEFAULT_BACKUP_SCHEDULE_TYPE_UNSPECIFIED  `

Not specified.

`  NONE  `

A default backup schedule isn't created automatically when a new database is created in the instance.

`  AUTOMATIC  `

A default backup schedule is created automatically when a new database is created in the instance. The default backup schedule creates a full backup every 24 hours. These full backups are retained for 7 days. You can edit or delete the default backup schedule once it's created.

## Edition

The edition selected for this instance. Different editions provide different capabilities at different price points.

Enums

`  EDITION_UNSPECIFIED  `

Edition not specified.

`  STANDARD  `

Standard edition.

`  ENTERPRISE  `

Enterprise edition.

`  ENTERPRISE_PLUS  `

Enterprise Plus edition.

## InstanceType

The type of this instance. The type can be used to distinguish product variants, that can affect aspects like: usage restrictions, quotas and billing. Currently this is used to distinguish FREE\_INSTANCE vs PROVISIONED instances.

Enums

`  INSTANCE_TYPE_UNSPECIFIED  `

Not specified.

`  PROVISIONED  `

Provisioned instances have dedicated resources, standard usage limits and support.

`  FREE_INSTANCE  `

Free instances provide no guarantee for dedicated resources, \[node\_count, processing\_units\] should be 0. They come with stricter usage limits and limited support.

## State

Indicates the current state of the instance.

Enums

`  STATE_UNSPECIFIED  `

Not specified.

`  CREATING  `

The instance is still being created. Resources may not be available yet, and operations such as database creation may not work.

`  READY  `

The instance is fully created and ready to do work such as creating databases.

## InstanceConfig

A possible configuration for a Cloud Spanner instance. Configurations define the geographic placement of nodes and their replication.

Fields

`  name  `

`  string  `

A unique identifier for the instance configuration. Values are of the form `  projects/<project>/instanceConfigs/[a-z][-a-z0-9]*  ` .

User instance configuration must start with `  custom-  ` .

`  display_name  `

`  string  `

The name of this instance configuration as it appears in UIs.

`  config_type  `

`  Type  `

Output only. Whether this instance configuration is a Google-managed or user-managed configuration.

`  replicas[]  `

`  ReplicaInfo  `

The geographic placement of nodes in this instance configuration and their replication properties.

To create user-managed configurations, input `  replicas  ` must include all replicas in `  replicas  ` of the `  base_config  ` and include one or more replicas in the `  optional_replicas  ` of the `  base_config  ` .

`  optional_replicas[]  `

`  ReplicaInfo  `

Output only. The available optional replicas to choose from for user-managed configurations. Populated for Google-managed configurations.

`  base_config  `

`  string  `

Base configuration name, e.g. projects/ /instanceConfigs/nam3, based on which this configuration is created. Only set for user-managed configurations. `  base_config  ` must refer to a configuration of type `  GOOGLE_MANAGED  ` in the same project as this configuration.

`  labels  `

`  map<string, string>  `

Cloud Labels are a flexible and lightweight mechanism for organizing cloud resources into groups that reflect a customer's organizational needs and deployment strategies. Cloud Labels can be used to filter collections of resources. They can be used to control how resource metrics are aggregated. And they can be used as arguments to policy management rules (e.g. route, firewall, load balancing, etc.).

  - Label keys must be between 1 and 63 characters long and must conform to the following regular expression: `  [a-z][a-z0-9_-]{0,62}  ` .
  - Label values must be between 0 and 63 characters long and must conform to the regular expression `  [a-z0-9_-]{0,63}  ` .
  - No more than 64 labels can be associated with a given resource.

See <https://goo.gl/xmQnxf> for more information on and examples of labels.

If you plan to use labels in your own code, please note that additional characters may be allowed in the future. Therefore, you are advised to use an internal label representation, such as JSON, which doesn't rely upon specific characters being disallowed. For example, representing labels as the string: name + "\_" + value would prove problematic if we were to allow "\_" in a future release.

`  etag  `

`  string  `

etag is used for optimistic concurrency control as a way to help prevent simultaneous updates of a instance configuration from overwriting each other. It is strongly suggested that systems make use of the etag in the read-modify-write cycle to perform instance configuration updates in order to avoid race conditions: An etag is returned in the response which contains instance configurations, and systems are expected to put that etag in the request to update instance configuration to ensure that their change is applied to the same version of the instance configuration. If no etag is provided in the call to update the instance configuration, then the existing instance configuration is overwritten blindly.

`  leader_options[]  `

`  string  `

Allowed values of the "default\_leader" schema option for databases in instances that use this instance configuration.

`  reconciling  `

`  bool  `

Output only. If true, the instance configuration is being created or updated. If false, there are no ongoing operations for the instance configuration.

`  state  `

`  State  `

Output only. The current instance configuration state. Applicable only for `  USER_MANAGED  ` configurations.

`  free_instance_availability  `

`  FreeInstanceAvailability  `

Output only. Describes whether free instances are available to be created in this instance configuration.

`  quorum_type  `

`  QuorumType  `

Output only. The `  QuorumType  ` of the instance configuration.

`  storage_limit_per_processing_unit  `

`  int64  `

Output only. The storage limit in bytes per processing unit.

## FreeInstanceAvailability

Describes the availability for free instances to be created in an instance configuration.

Enums

`  FREE_INSTANCE_AVAILABILITY_UNSPECIFIED  `

Not specified.

`  AVAILABLE  `

Indicates that free instances are available to be created in this instance configuration.

`  UNSUPPORTED  `

Indicates that free instances are not supported in this instance configuration.

`  DISABLED  `

Indicates that free instances are currently not available to be created in this instance configuration.

`  QUOTA_EXCEEDED  `

Indicates that additional free instances cannot be created in this instance configuration because the project has reached its limit of free instances.

## QuorumType

Indicates the quorum type of this instance configuration.

Enums

`  QUORUM_TYPE_UNSPECIFIED  `

Quorum type not specified.

`  REGION  `

An instance configuration tagged with `  REGION  ` quorum type forms a write quorum in a single region.

`  DUAL_REGION  `

An instance configuration tagged with the `  DUAL_REGION  ` quorum type forms a write quorum with exactly two read-write regions in a multi-region configuration.

This instance configuration requires failover in the event of regional failures.

`  MULTI_REGION  `

An instance configuration tagged with the `  MULTI_REGION  ` quorum type forms a write quorum from replicas that are spread across more than one region in a multi-region configuration.

## State

Indicates the current state of the instance configuration.

Enums

`  STATE_UNSPECIFIED  `

Not specified.

`  CREATING  `

The instance configuration is still being created.

`  READY  `

The instance configuration is fully created and ready to be used to create instances.

## Type

The type of this configuration.

Enums

`  TYPE_UNSPECIFIED  `

Unspecified.

`  GOOGLE_MANAGED  `

Google-managed configuration.

`  USER_MANAGED  `

User-managed configuration.

## InstancePartition

An isolated set of Cloud Spanner resources that databases can define placements on.

Fields

`  name  `

`  string  `

Required. A unique identifier for the instance partition. Values are of the form `  projects/<project>/instances/<instance>/instancePartitions/[a-z][-a-z0-9]*[a-z0-9]  ` . The final segment of the name must be between 2 and 64 characters in length. An instance partition's name cannot be changed after the instance partition is created.

`  config  `

`  string  `

Required. The name of the instance partition's configuration. Values are of the form `  projects/<project>/instanceConfigs/<configuration>  ` . See also `  InstanceConfig  ` and `  ListInstanceConfigs  ` .

`  display_name  `

`  string  `

Required. The descriptive name for this instance partition as it appears in UIs. Must be unique per project and between 4 and 30 characters in length.

`  state  `

`  State  `

Output only. The current instance partition state.

`  create_time  `

`  Timestamp  `

Output only. The time at which the instance partition was created.

`  update_time  `

`  Timestamp  `

Output only. The time at which the instance partition was most recently updated.

`  referencing_databases[]  `

`  string  `

Output only. The names of the databases that reference this instance partition. Referencing databases should share the parent instance. The existence of any referencing database prevents the instance partition from being deleted.

`  referencing_backups[] (deprecated)  `

`  string  `

This item is deprecated\!

Output only. Deprecated: This field is not populated. Output only. The names of the backups that reference this instance partition. Referencing backups should share the parent instance. The existence of any referencing backup prevents the instance partition from being deleted.

`  etag  `

`  string  `

Used for optimistic concurrency control as a way to help prevent simultaneous updates of a instance partition from overwriting each other. It is strongly suggested that systems make use of the etag in the read-modify-write cycle to perform instance partition updates in order to avoid race conditions: An etag is returned in the response which contains instance partitions, and systems are expected to put that etag in the request to update instance partitions to ensure that their change will be applied to the same version of the instance partition. If no etag is provided in the call to update instance partition, then the existing instance partition is overwritten blindly.

Union field `  compute_capacity  ` . Compute capacity defines amount of server and storage resources that are available to the databases in an instance partition. At most, one of either `  node_count  ` or `  processing_units  ` should be present in the message. For more information, see [Compute capacity, nodes, and processing units](https://cloud.google.com/spanner/docs/compute-capacity) . `  compute_capacity  ` can be only one of the following:

`  node_count  `

`  int32  `

The number of nodes allocated to this instance partition.

Users can set the `  node_count  ` field to specify the target number of nodes allocated to the instance partition.

This may be zero in API responses for instance partitions that are not yet in state `  READY  ` .

`  processing_units  `

`  int32  `

The number of processing units allocated to this instance partition.

Users can set the `  processing_units  ` field to specify the target number of processing units allocated to the instance partition.

This might be zero in API responses for instance partitions that are not yet in the `  READY  ` state.

## State

Indicates the current state of the instance partition.

Enums

`  STATE_UNSPECIFIED  `

Not specified.

`  CREATING  `

The instance partition is still being created. Resources may not be available yet, and operations such as creating placements using this instance partition may not work.

`  READY  `

The instance partition is fully created and ready to do work such as creating placements and using in databases.

## ListInstanceConfigOperationsRequest

The request for `  ListInstanceConfigOperations  ` .

Fields

`  parent  `

`  string  `

Required. The project of the instance configuration operations. Values are of the form `  projects/<project>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  parent  ` :

  - `  spanner.instanceConfigOperations.list  `

`  filter  `

`  string  `

An expression that filters the list of returned operations.

A filter expression consists of a field name, a comparison operator, and a value for filtering. The value must be a string, a number, or a boolean. The comparison operator must be one of: `  <  ` , `  >  ` , `  <=  ` , `  >=  ` , `  !=  ` , `  =  ` , or `  :  ` . Colon `  :  ` is the contains operator. Filter rules are not case sensitive.

The following fields in the Operation are eligible for filtering:

  - `  name  ` - The name of the long-running operation
  - `  done  ` - False if the operation is in progress, else true.
  - `  metadata.@type  ` - the type of metadata. For example, the type string for `  CreateInstanceConfigMetadata  ` is `  type.googleapis.com/google.spanner.admin.instance.v1.CreateInstanceConfigMetadata  ` .
  - `  metadata.<field_name>  ` - any field in metadata.value. `  metadata.@type  ` must be specified first, if filtering on metadata fields.
  - `  error  ` - Error associated with the long-running operation.
  - `  response.@type  ` - the type of response.
  - `  response.<field_name>  ` - any field in response.value.

You can combine multiple expressions by enclosing each expression in parentheses. By default, expressions are combined with AND logic. However, you can specify AND, OR, and NOT logic explicitly.

Here are a few examples:

  - `  done:true  ` - The operation is complete.
  - `  (metadata.@type=  ` \\ `  type.googleapis.com/google.spanner.admin.instance.v1.CreateInstanceConfigMetadata) AND  ` \\ `  (metadata.instance_config.name:custom-config) AND  ` \\ `  (metadata.progress.start_time < \"2021-03-28T14:50:00Z\") AND  ` \\ `  (error:*)  ` - Return operations where:
      - The operation's metadata type is `  CreateInstanceConfigMetadata  ` .
      - The instance configuration name contains "custom-config".
      - The operation started before 2021-03-28T14:50:00Z.
      - The operation resulted in an error.

`  page_size  `

`  int32  `

Number of operations to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.

`  page_token  `

`  string  `

If non-empty, `  page_token  ` should contain a `  next_page_token  ` from a previous `  ListInstanceConfigOperationsResponse  ` to the same `  parent  ` and with the same `  filter  ` .

## ListInstanceConfigOperationsResponse

The response for `  ListInstanceConfigOperations  ` .

Fields

`  operations[]  `

`  Operation  `

The list of matching instance configuration long-running operations. Each operation's name will be prefixed by the name of the instance configuration. The operation's metadata field type `  metadata.type_url  ` describes the type of the metadata.

`  next_page_token  `

`  string  `

`  next_page_token  ` can be sent in a subsequent `  ListInstanceConfigOperations  ` call to fetch more of the matching metadata.

## ListInstanceConfigsRequest

The request for `  ListInstanceConfigs  ` .

Fields

`  parent  `

`  string  `

Required. The name of the project for which a list of supported instance configurations is requested. Values are of the form `  projects/<project>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  parent  ` :

  - `  spanner.instanceConfigs.list  `

`  page_size  `

`  int32  `

Number of instance configurations to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.

`  page_token  `

`  string  `

If non-empty, `  page_token  ` should contain a `  next_page_token  ` from a previous `  ListInstanceConfigsResponse  ` .

## ListInstanceConfigsResponse

The response for `  ListInstanceConfigs  ` .

Fields

`  instance_configs[]  `

`  InstanceConfig  `

The list of requested instance configurations.

`  next_page_token  `

`  string  `

`  next_page_token  ` can be sent in a subsequent `  ListInstanceConfigs  ` call to fetch more of the matching instance configurations.

## ListInstancePartitionOperationsRequest

The request for `  ListInstancePartitionOperations  ` .

Fields

`  parent  `

`  string  `

Required. The parent instance of the instance partition operations. Values are of the form `  projects/<project>/instances/<instance>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  parent  ` :

  - `  spanner.instancePartitionOperations.list  `

`  filter  `

`  string  `

Optional. An expression that filters the list of returned operations.

A filter expression consists of a field name, a comparison operator, and a value for filtering. The value must be a string, a number, or a boolean. The comparison operator must be one of: `  <  ` , `  >  ` , `  <=  ` , `  >=  ` , `  !=  ` , `  =  ` , or `  :  ` . Colon `  :  ` is the contains operator. Filter rules are not case sensitive.

The following fields in the Operation are eligible for filtering:

  - `  name  ` - The name of the long-running operation
  - `  done  ` - False if the operation is in progress, else true.
  - `  metadata.@type  ` - the type of metadata. For example, the type string for `  CreateInstancePartitionMetadata  ` is `  type.googleapis.com/google.spanner.admin.instance.v1.CreateInstancePartitionMetadata  ` .
  - `  metadata.<field_name>  ` - any field in metadata.value. `  metadata.@type  ` must be specified first, if filtering on metadata fields.
  - `  error  ` - Error associated with the long-running operation.
  - `  response.@type  ` - the type of response.
  - `  response.<field_name>  ` - any field in response.value.

You can combine multiple expressions by enclosing each expression in parentheses. By default, expressions are combined with AND logic. However, you can specify AND, OR, and NOT logic explicitly.

Here are a few examples:

  - `  done:true  ` - The operation is complete.
  - `  (metadata.@type=  ` \\ `  type.googleapis.com/google.spanner.admin.instance.v1.CreateInstancePartitionMetadata) AND  ` \\ `  (metadata.instance_partition.name:custom-instance-partition) AND  ` \\ `  (metadata.start_time < \"2021-03-28T14:50:00Z\") AND  ` \\ `  (error:*)  ` - Return operations where:
      - The operation's metadata type is `  CreateInstancePartitionMetadata  ` .
      - The instance partition name contains "custom-instance-partition".
      - The operation started before 2021-03-28T14:50:00Z.
      - The operation resulted in an error.

`  page_size  `

`  int32  `

Optional. Number of operations to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.

`  page_token  `

`  string  `

Optional. If non-empty, `  page_token  ` should contain a `  next_page_token  ` from a previous `  ListInstancePartitionOperationsResponse  ` to the same `  parent  ` and with the same `  filter  ` .

`  instance_partition_deadline  `

`  Timestamp  `

Optional. Deadline used while retrieving metadata for instance partition operations. Instance partitions whose operation metadata cannot be retrieved within this deadline will be added to `  unreachable_instance_partitions  ` in `  ListInstancePartitionOperationsResponse  ` .

## ListInstancePartitionOperationsResponse

The response for `  ListInstancePartitionOperations  ` .

Fields

`  operations[]  `

`  Operation  `

The list of matching instance partition long-running operations. Each operation's name will be prefixed by the instance partition's name. The operation's metadata field type `  metadata.type_url  ` describes the type of the metadata.

`  next_page_token  `

`  string  `

`  next_page_token  ` can be sent in a subsequent `  ListInstancePartitionOperations  ` call to fetch more of the matching metadata.

`  unreachable_instance_partitions[]  `

`  string  `

The list of unreachable instance partitions. It includes the names of instance partitions whose operation metadata could not be retrieved within `  instance_partition_deadline  ` .

## ListInstancePartitionsRequest

The request for `  ListInstancePartitions  ` .

Fields

`  parent  `

`  string  `

Required. The instance whose instance partitions should be listed. Values are of the form `  projects/<project>/instances/<instance>  ` . Use `  {instance} = '-'  ` to list instance partitions for all Instances in a project, e.g., `  projects/myproject/instances/-  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  parent  ` :

  - `  spanner.instancePartitions.list  `

`  page_size  `

`  int32  `

Number of instance partitions to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.

`  page_token  `

`  string  `

If non-empty, `  page_token  ` should contain a `  next_page_token  ` from a previous `  ListInstancePartitionsResponse  ` .

`  instance_partition_deadline  `

`  Timestamp  `

Optional. Deadline used while retrieving metadata for instance partitions. Instance partitions whose metadata cannot be retrieved within this deadline will be added to `  unreachable  ` in `  ListInstancePartitionsResponse  ` .

## ListInstancePartitionsResponse

The response for `  ListInstancePartitions  ` .

Fields

`  instance_partitions[]  `

`  InstancePartition  `

The list of requested instancePartitions.

`  next_page_token  `

`  string  `

`  next_page_token  ` can be sent in a subsequent `  ListInstancePartitions  ` call to fetch more of the matching instance partitions.

`  unreachable[]  `

`  string  `

The list of unreachable instances or instance partitions. It includes the names of instances or instance partitions whose metadata could not be retrieved within `  instance_partition_deadline  ` .

## ListInstancesRequest

The request for `  ListInstances  ` .

Fields

`  parent  `

`  string  `

Required. The name of the project for which a list of instances is requested. Values are of the form `  projects/<project>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  parent  ` :

  - `  spanner.instances.list  `

`  page_size  `

`  int32  `

Number of instances to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.

`  page_token  `

`  string  `

If non-empty, `  page_token  ` should contain a `  next_page_token  ` from a previous `  ListInstancesResponse  ` .

`  filter  `

`  string  `

An expression for filtering the results of the request. Filter rules are case insensitive. The fields eligible for filtering are:

  - `  name  `
  - `  display_name  `
  - `  labels.key  ` where key is the name of a label

Some examples of using filters are:

  - `  name:*  ` --\> The instance has a name.
  - `  name:Howl  ` --\> The instance's name contains the string "howl".
  - `  name:HOWL  ` --\> Equivalent to above.
  - `  NAME:howl  ` --\> Equivalent to above.
  - `  labels.env:*  ` --\> The instance has the label "env".
  - `  labels.env:dev  ` --\> The instance has the label "env" and the value of the label contains the string "dev".
  - `  name:howl labels.env:dev  ` --\> The instance's name contains "howl" and it has the label "env" with its value containing "dev".

`  instance_deadline  `

`  Timestamp  `

Deadline used while retrieving metadata for instances. Instances whose metadata cannot be retrieved within this deadline will be added to `  unreachable  ` in `  ListInstancesResponse  ` .

## ListInstancesResponse

The response for `  ListInstances  ` .

Fields

`  instances[]  `

`  Instance  `

The list of requested instances.

`  next_page_token  `

`  string  `

`  next_page_token  ` can be sent in a subsequent `  ListInstances  ` call to fetch more of the matching instances.

`  unreachable[]  `

`  string  `

The list of unreachable instances. It includes the names of instances whose metadata could not be retrieved within `  instance_deadline  ` .

## MoveInstanceMetadata

Metadata type for the operation returned by `  MoveInstance  ` .

Fields

`  target_config  `

`  string  `

The target instance configuration where to move the instance. Values are of the form `  projects/<project>/instanceConfigs/<config>  ` .

`  progress  `

`  OperationProgress  `

The progress of the `  MoveInstance  ` operation. `  progress_percent  ` is reset when cancellation is requested.

`  cancel_time  `

`  Timestamp  `

The time at which this operation was cancelled.

## MoveInstanceRequest

The request for `  MoveInstance  ` .

Fields

`  name  `

`  string  `

Required. The instance to move. Values are of the form `  projects/<project>/instances/<instance>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  name  ` :

  - `  spanner.instances.update  `

`  target_config  `

`  string  `

Required. The target instance configuration where to move the instance. Values are of the form `  projects/<project>/instanceConfigs/<config>  ` .

`  target_database_move_configs[]  `

`  DatabaseMoveConfig  `

Optional. The configuration for each database in the target instance configuration.

## DatabaseMoveConfig

The configuration for each database in the target instance configuration.

Fields

`  database_id  `

`  string  `

Required. The unique identifier of the database resource in the Instance. For example, if the database uri is `  projects/foo/instances/bar/databases/baz  ` , then the id to supply here is baz.

`  encryption_config  `

`  EncryptionConfig  `

Optional. Encryption configuration to be used for the database in the target configuration. The encryption configuration must be specified for every database which currently uses CMEK encryption. If a database currently uses Google-managed encryption and a target encryption configuration is not specified, then the database defaults to Google-managed encryption.

If a database currently uses Google-managed encryption and a target CMEK encryption is specified, the request is rejected.

If a database currently uses CMEK encryption, then a target encryption configuration must be specified. You can't move a CMEK database to a Google-managed encryption database using the MoveInstance API.

## MoveInstanceResponse

This type has no fields.

The response for `  MoveInstance  ` .

## OperationProgress

Encapsulates progress related information for a Cloud Spanner long running instance operations.

Fields

`  progress_percent  `

`  int32  `

Percent completion of the operation. Values are between 0 and 100 inclusive.

`  start_time  `

`  Timestamp  `

Time the request was received.

`  end_time  `

`  Timestamp  `

If set, the time at which this operation failed or was completed successfully.

## ReplicaComputeCapacity

ReplicaComputeCapacity describes the amount of server resources that are allocated to each replica identified by the replica selection.

Fields

`  replica_selection  `

`  ReplicaSelection  `

Required. Identifies replicas by specified properties. All replicas in the selection have the same amount of compute capacity.

Union field `  compute_capacity  ` . Compute capacity allocated to each replica identified by the specified selection. The unit is selected based on the unit used to specify the instance size for non-autoscaling instances, or the unit used in autoscaling limit for autoscaling instances. `  compute_capacity  ` can be only one of the following:

`  node_count  `

`  int32  `

The number of nodes allocated to each replica.

This may be zero in API responses for instances that are not yet in state `  READY  ` .

`  processing_units  `

`  int32  `

The number of processing units allocated to each replica.

This may be zero in API responses for instances that are not yet in state `  READY  ` .

## ReplicaInfo

Fields

`  location  `

`  string  `

The location of the serving resources, e.g., "us-central1".

`  type  `

`  ReplicaType  `

The type of replica.

`  default_leader_location  `

`  bool  `

If true, this location is designated as the default leader location where leader replicas are placed. See the [region types documentation](https://cloud.google.com/spanner/docs/instances#region_types) for more details.

## ReplicaType

Indicates the type of replica. See the [replica types documentation](https://cloud.google.com/spanner/docs/replication#replica_types) for more details.

Enums

`  TYPE_UNSPECIFIED  `

Not specified.

`  READ_WRITE  `

Read-write replicas support both reads and writes. These replicas:

  - Maintain a full copy of your data.
  - Serve reads.
  - Can vote whether to commit a write.
  - Participate in leadership election.
  - Are eligible to become a leader.

`  READ_ONLY  `

Read-only replicas only support reads (not writes). Read-only replicas:

  - Maintain a full copy of your data.
  - Serve reads.
  - Do not participate in voting to commit writes.
  - Are not eligible to become a leader.

`  WITNESS  `

Witness replicas don't support reads but do participate in voting to commit writes. Witness replicas:

  - Do not maintain a full copy of data.
  - Do not serve reads.
  - Vote whether to commit writes.
  - Participate in leader election but are not eligible to become leader.

## ReplicaSelection

ReplicaSelection identifies replicas with common properties.

Fields

`  location  `

`  string  `

Required. Name of the location of the replicas (for example, "us-central1").

## UpdateInstanceConfigMetadata

Metadata type for the operation returned by `  UpdateInstanceConfig  ` .

Fields

`  instance_config  `

`  InstanceConfig  `

The desired instance configuration after updating.

`  progress  `

`  OperationProgress  `

The progress of the `  UpdateInstanceConfig  ` operation.

`  cancel_time  `

`  Timestamp  `

The time at which this operation was cancelled.

## UpdateInstanceConfigRequest

The request for `  UpdateInstanceConfig  ` .

Fields

`  instance_config  `

`  InstanceConfig  `

Required. The user instance configuration to update, which must always include the instance configuration name. Otherwise, only fields mentioned in `  update_mask  ` need be included. To prevent conflicts of concurrent updates, `  etag  ` can be used.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  instanceConfig  ` :

  - `  spanner.instanceConfigs.update  `

`  update_mask  `

`  FieldMask  `

Required. A mask specifying which fields in `  InstanceConfig  ` should be updated. The field mask must always be specified; this prevents any future fields in `  InstanceConfig  ` from being erased accidentally by clients that do not know about them. Only display\_name and labels can be updated.

`  validate_only  `

`  bool  `

An option to validate, but not actually execute, a request, and provide the same response.

## UpdateInstanceMetadata

Metadata type for the operation returned by `  UpdateInstance  ` .

Fields

`  instance  `

`  Instance  `

The desired end state of the update.

`  start_time  `

`  Timestamp  `

The time at which `  UpdateInstance  ` request was received.

`  cancel_time  `

`  Timestamp  `

The time at which this operation was cancelled. If set, this operation is in the process of undoing itself (which is guaranteed to succeed) and cannot be cancelled again.

`  end_time  `

`  Timestamp  `

The time at which this operation failed or was completed successfully.

`  expected_fulfillment_period  `

`  FulfillmentPeriod  `

The expected fulfillment period of this update operation.

## UpdateInstancePartitionMetadata

Metadata type for the operation returned by `  UpdateInstancePartition  ` .

Fields

`  instance_partition  `

`  InstancePartition  `

The desired end state of the update.

`  start_time  `

`  Timestamp  `

The time at which `  UpdateInstancePartition  ` request was received.

`  cancel_time  `

`  Timestamp  `

The time at which this operation was cancelled. If set, this operation is in the process of undoing itself (which is guaranteed to succeed) and cannot be cancelled again.

`  end_time  `

`  Timestamp  `

The time at which this operation failed or was completed successfully.

## UpdateInstancePartitionRequest

The request for `  UpdateInstancePartition  ` .

Fields

`  instance_partition  `

`  InstancePartition  `

Required. The instance partition to update, which must always include the instance partition name. Otherwise, only fields mentioned in `  field_mask  ` need be included.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  instancePartition  ` :

  - `  spanner.instancePartitions.update  `

`  field_mask  `

`  FieldMask  `

Required. A mask specifying which fields in `  InstancePartition  ` should be updated. The field mask must always be specified; this prevents any future fields in `  InstancePartition  ` from being erased accidentally by clients that do not know about them.

## UpdateInstanceRequest

The request for `  UpdateInstance  ` .

Fields

`  instance  `

`  Instance  `

Required. The instance to update, which must always include the instance name. Otherwise, only fields mentioned in `  field_mask  ` need be included.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  instance  ` :

  - `  spanner.instances.update  `

`  field_mask  `

`  FieldMask  `

Required. A mask specifying which fields in `  Instance  ` should be updated. The field mask must always be specified; this prevents any future fields in `  Instance  ` from being erased accidentally by clients that do not know about them.
