---
name: documents/docs.cloud.google.com/spanner-omni/deployment-configuration
uri: https://docs.cloud.google.com/spanner-omni/deployment-configuration
title: Deployment configurations
description: A downloadable, self-managed version of Spanner.
data_source: docs.cloud.google.com
---

This document describes deployment configurations for Spanner Omni on virtual machines (VMs) or bare-metal servers. It explains the structure and configuration options for the YAML deployment configuration file ( `deployment.yaml` ) used to define VM deployment topologies and runtime parameters when using the Spanner Omni CLI.

To learn how to create a deployment, see one of the following:

  - [Create a deployment on VMs](https://docs.cloud.google.com/spanner-omni/deploy-on-vms)
  - [Create a secure deployment on VMs](https://docs.cloud.google.com/spanner-omni/deploy-encryption-vms)
  - [Create a deployment on Kubernetes](https://docs.cloud.google.com/spanner-omni/deploy-on-kubernetes)
  - [Create a secure deployment on Kubernetes](https://docs.cloud.google.com/spanner-omni/deploy-encryption-kubernetes)

## Deployment configuration overview

When you create a deployment on VMs or bare-metal servers, you pass this configuration file to the `spanner deployment create` command in the Spanner Omni CLI:

    spanner deployment create --config-file=deployment.yaml

The deployment configuration defines the following key elements:

  - **[Single-server mode](https://docs.cloud.google.com/spanner-omni/deployment-configuration#single-server)** : Optimization mode that restricts the entire deployment to a single server for development and testing.
  - **[Locations](https://docs.cloud.google.com/spanner-omni/deployment-configuration#locations)** : Physical sites or cloud regions where your servers reside.
  - **[Location distances](https://docs.cloud.google.com/spanner-omni/deployment-configuration#location-distances)** : Network latencies between pairs of locations.
  - **[Zones](https://docs.cloud.google.com/spanner-omni/deployment-configuration#zones)** : Logical groupings of servers that represent [Paxos replicas](https://docs.cloud.google.com/spanner-omni/key-terms#replica) .
  - **[Root servers](https://docs.cloud.google.com/spanner-omni/deployment-configuration#root-servers)** : Dedicated servers responsible for zone metadata and membership quorum.
  - **[Replica types](https://docs.cloud.google.com/spanner-omni/deployment-configuration#replica-types)** : Roles for each zone (read-write, witness, or read-only).
  - **[Clock SLA](https://docs.cloud.google.com/spanner-omni/deployment-configuration#clock-sla)** : TrueTime synchronization parameters, including clock jitter and drift rate error.
  - **[Deployment settings](https://docs.cloud.google.com/spanner-omni/deployment-configuration#deployment-settings)** : Global settings such as preferred leader locations and authentication security settings.

## Configuration file structure

The following example shows the top-level structure of a deployment configuration file:

    # Deployment name
    name: regional-deployment
    
    # Restrict the entire deployment to a single server (optional, default: false)
    single_server: false
    
    # Physical or logical locations (regions)
    location:
      - name: us-central1
    
    # Network distances between locations (optional)
    location_distance:
      - src: us-central1
        dest: us-east1
        latency_ms: 30
    
    # Zones and root servers in the deployment
    zone:
      - name: us-central1-a
        location: us-central1
        single_server: false
        replica_type: READ_WRITE
        root_server:
          - host: rootserver1.example.internal
            port_base: 15000
    
    # Clock synchronization SLA parameters (optional)
    clock_sla:
      jitter_in_s: 0.005
      rate_error_in_ppm: 200
    
    # Deployment settings (optional)
    deployment_settings:
      preferred_leader_location: us-central1
      security_settings:
        insecure_mode: true

## Top-level fields

The deployment configuration supports the following top-level fields:

| Field                                  | Type            | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| -------------------------------------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `         name        `                | String          | The name of the deployment, such as `prod` , `staging` , or `regional-deployment` .                                                                                                                                                                                                                                                                                                                                                                                            |
| `         single_server        `       | Boolean         | Optional. If set to `true` , specifies that the entire deployment is a single-server deployment, restricting it to one zone and one server. Deployments created with `single_server: true` can't add zones or servers after creation. If you want to run Spanner Omni in single-server mode, you don't need to manually create this configuration because Spanner Omni automatically generates it when you run the `spanner start-single-server` command. Default is `false` . |
| `         location        `            | List of objects | The physical or logical locations (regions) in the deployment.                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `         location_distance        `   | List of objects | Optional. The network latency between pairs of locations.                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `         zone        `                | List of objects | Required. The zones that make up the deployment. You must specify at least one zone.                                                                                                                                                                                                                                                                                                                                                                                           |
| `         clock_sla        `           | Object          | Optional. The clock synchronization service level agreement (SLA) parameters for software TrueTime.                                                                                                                                                                                                                                                                                                                                                                            |
| `         deployment_settings        ` | Object          | Optional. Runtime settings for preferred leader placement and security authentication.                                                                                                                                                                                                                                                                                                                                                                                         |

## Deployment name

The `name` field specifies a user-chosen name for the deployment. You can use any string that identifies the deployment, such as `prod` , `staging` , or `regional-deployment` .

## Single-server mode

The top-level `single_server` field specifies that the entire deployment is a single-server deployment. When set to `true` , this setting restricts the deployment to one zone and one server, which reduces resource overhead for local development and testing environments. Deployments created with `single_server:true` can't add zones or servers after creation.

If you want to run Spanner Omni in single-server mode, you don't need to manually create this configuration. When you run the `spanner start-single-server` command, Spanner Omni automatically generates this configuration for you. For more information, see [Option A: Single-server deployment](https://docs.cloud.google.com/spanner-omni/deploy-on-vms#option-a-single-server) .

The top-level `single_server` field is distinct from the zone-level `single_server` field:

  - The **top-level** `single_server` field applies to the entire deployment.
  - The **zone-level** `single_server` field applies only to an individual zone within a deployment. For more information, see [Single-server zones](https://docs.cloud.google.com/spanner-omni/deployment-configuration#single-server-zones) .

## Locations

A location represents a physical data center or cloud region where machines are located (equivalent to a region in Google Cloud).

You define locations in the `location` list:

    location:
      - name: us-central1
      - name: europe-west2

Location names must satisfy the following requirements:

  - Must start with a letter and end with a letter or digit.
  - Can contain only letters, digits, underscores ( `_` ), and dashes ( `-` ).
  - Can optionally include a domain prefix followed by a colon (for example, `cloud.google.com:us-east1` or `onprem:datacenter1` ).
  - Can't use the reserved name `default` .
  - Must be unique across the deployment.

## Location distances

The `location_distance` list specifies network latency between pairs of locations. Spanner Omni uses this information to optimize replication and query routing.

    location_distance:
      - src: us-central1
        dest: europe-west2
        latency_ms: 105
      - src: europe-west2
        dest: us-central1
        latency_ms: 110

Each location distance object contains the following fields:

  - **`src`** : Required. The name of the source location. Must match a defined location in the `location` list.
  - **`dest`** : Required. The name of the destination location. Must match a defined location and can't be identical to `src` .
  - **`latency_ms`** : The network latency in milliseconds. Must be a non-negative integer. If omitted, Spanner Omni assumes that latency is negligible (sub-millisecond).

Network latency in physical networks isn't always symmetric. If you provide both `(src, dest)` and `(dest, src)` , Spanner Omni honors both measurements. If you provide only one direction, Spanner Omni assumes the reverse direction has the same latency.

## Zones

A zone is a logical grouping of one or more servers within a location. For data replication, each zone represents a Paxos replica. A deployment must have at least one zone.

    zone:
      - name: us-central1-a
        location: us-central1
        single_server: false
        replica_type: READ_WRITE
        root_server:
          - host: rootserver1.example.internal
            port_base: 15000
          - host: rootserver2.example.internal
            port_base: 15000
          - host: rootserver3.example.internal
            port_base: 15000

Each zone object supports the following fields:

| Field                            | Type            | Description                                                                                                                                                                                                                                                                                                                                                                                                       |
| -------------------------------- | --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`                           | String          | Required. The name of the zone. Follows the same naming rules as location names. Must be unique across the deployment.                                                                                                                                                                                                                                                                                            |
| `location`                       | String          | The name of the location where the zone resides. Must match a defined location in the `location` list. If omitted, Spanner Omni assigns the zone to the `default` location.                                                                                                                                                                                                                                       |
| `         single_server        ` | Boolean         | Optional. If set to `true` , designates that this zone has only a single server (can only ever have one root server and no other servers). Eliminates zone metadata replication overhead within the zone. In a multi-zone deployment, you can set this to `true` for specific zones, such as a `WITNESS` replica zone that doesn't store user data, while other zones have multiple servers. Default is `false` . |
| `replica_type`                   | Enum string     | The replica role of the zone in Paxos quorums. Supported values are `READ_WRITE` , `WITNESS` , and `READ_ONLY` . Default is `READ_WRITE` .                                                                                                                                                                                                                                                                        |
| `root_server`                    | List of objects | Required. The list of root servers in the zone.                                                                                                                                                                                                                                                                                                                                                                   |

### Replica types

Spanner Omni supports three replica types for zones:

  - **`READ_WRITE`** : Stores a complete copy of user data, serves read requests, and votes in Paxos quorums. Read-write replicas are eligible to become Paxos leaders to propose writes.
  - **`WITNESS`** : Votes in Paxos quorums to help achieve consensus, but can't become a leader. Witness replicas don't store user data and can't serve read requests. They help achieve quorum without the storage overhead or write latency of a full replica across distant locations.
  - **`READ_ONLY`** : Stores a complete copy of user data that is asynchronously replicated from leaders. Read-only replicas can't become leaders and don't vote in Paxos quorums. They offload read traffic from read-write replicas.

When configuring replica types, ensure your deployment satisfies the following rules:

  - The deployment must contain at least one `READ_WRITE` zone.
  - The number of `READ_WRITE` zones must be strictly greater than the number of `WITNESS` zones.

### Root servers

Root servers have special responsibilities in Spanner Omni. They store zone metadata and manage membership for other servers in the zone. If a quorum of root servers becomes unavailable, the entire zone becomes unavailable.

When configuring root servers in `deployment.yaml` , keep the following guidelines in mind:

  - The number of root servers per zone must be an odd number between one and nine, inclusive, to ensure quorum for consistency. If the number of servers is an even number, deployments might fail. When configuring your zones, designate servers as root servers. We recommend that you use one for development or testing and three for highly available production zones.
  - Only specify root servers in the `deployment.yaml` file during initial deployment creation. Non-root servers can be added later to scale compute and storage capacity.

Each root server object supports the following fields:

  - **`host`** : Required. The hostname or IP address of the machine running the server.
  - **`port_base`** : Optional. The starting port number for the server. Default is `15000` . This port becomes the public gRPC port for client connections. You must reserve ports in the range `[port_base + 1, port_base + 31]` (for example, `15001` through `15031` ) for internal Spanner Omni processes.

### Single-server zones

The zone-level `single_server` field specifies that an individual zone contains only a single server. A single-server zone can have only one root server and can't have additional servers added later. This setting eliminates the overhead of replicating zone metadata within that zone.

Unlike the [top-level `single_server` field](https://docs.cloud.google.com/spanner-omni/deployment-configuration#single-server) , which designates that the entire deployment consists of a single server, the zone-level `single_server` field applies only to that specific zone.

In a multi-zone deployment, you can configure individual zones as single-server zones while other zones contain multiple servers. For example, consider a deployment with two `READ_WRITE` replica zones and one `WITNESS` replica zone:

  - The two `READ_WRITE` zones contain multiple servers ( `single_server: false` ) to provide high availability and scale compute and storage capacity for user data.
  - You can configure the `WITNESS` zone as either a single-server zone or a multi-server zone, depending on your Paxos voting volume:
      - **Small to medium workloads** : If a single VM or server has sufficient capacity to process all Paxos voting traffic for the deployment, set `single_server: true` . Because witness replicas only cast votes and don't store user data, using a single server eliminates intra-zone metadata replication overhead.
      - **Large-scale deployments** : If you have high write throughput or many servers in each `READ_WRITE` zone (for example, dozens or hundreds of nodes), a single server can become overloaded and cause Paxos consensus failures. Configure the `WITNESS` zone with multiple servers ( `single_server: false` ) to distribute the voting workload.

For a configuration example, see [Multi-location deployment with witness replica](https://docs.cloud.google.com/spanner-omni/deployment-configuration#example-multi-location) .

## Clock SLA

Spanner Omni relies on software TrueTime to provide external consistency without requiring specialized GPS hardware or atomic clocks. The `clock_sla` object defines the expected synchronization bounds for server clocks across the deployment:

    clock_sla:
      jitter_in_s: 0.005
      rate_error_in_ppm: 200

The `clock_sla` configuration includes the following fields:

  - **`jitter_in_s`** : The maximum expected clock jitter in seconds. Must be a non-negative floating-point number ( `>= 0` ).
  - **`rate_error_in_ppm`** : The maximum clock drift rate error in parts per million (ppm). Must be a value between `0` and `10000` .

For more information about time synchronization, see [TrueTime and external consistency](https://docs.cloud.google.com/spanner-omni/true-time-external-consistency) .

## Deployment settings

The `deployment_settings` object configures global deployment behavior, including leader location preference and network security:

    deployment_settings:
      preferred_leader_location: us-central1
      security_settings:
        insecure_mode: false
        authentication_methods:
          - AUTHENTICATION_METHOD_PASSWORD
          - AUTHENTICATION_METHOD_CLIENT_CERTIFICATE
        password_authentication_protocol: PASSWORD_AUTHENTICATION_PROTOCOL_OPAQUE

### Preferred leader location

The `preferred_leader_location` field designates a location where Paxos leaders are preferentially placed. Electing leaders near your primary application workload reduces write latency by avoiding extra network round trips.

When configuring `preferred_leader_location` , ensure the following:

  - The specified location must match a defined location in the `location` list (or `default` ).
  - The designated location must contain at least one `READ_WRITE` zone.

### Security settings

The `security_settings` object configures authentication and encryption modes:

  - **`insecure_mode`** : Boolean. If set to `true` , disables authentication and authorization for incoming connections. This mode is intended for prototyping and evaluation only. Default is `false` .
  - **`authentication_methods`** : List of enabled authentication methods. Required if `insecure_mode` is `false` . Supported values:
      - `AUTHENTICATION_METHOD_PASSWORD` : Enables username and password authentication.
      - `AUTHENTICATION_METHOD_CLIENT_CERTIFICATE` : Enables mutual TLS (mTLS) client certificate authentication.
  - **`password_authentication_protocol`** : The protocol used for password verification. Required if `AUTHENTICATION_METHOD_PASSWORD` is included in `authentication_methods` . Supported value:
      - `PASSWORD_AUTHENTICATION_PROTOCOL_OPAQUE` : Uses the OPAQUE asymmetric password-authenticated key exchange protocol.

For more information about setting up encryption and credentials, see [Create a deployment with TLS encryption on VMs](https://docs.cloud.google.com/spanner-omni/deploy-encryption-vms) .

## Deployment configuration examples

The following examples demonstrate common deployment patterns.

### Regional multi-zone deployment

The following configuration creates a high-availability regional deployment across three zones in a single location:

    name: regional-prod
    location:
      - name: us-central1
    zone:
      - name: us-central1-a
        location: us-central1
        replica_type: READ_WRITE
        root_server:
          - host: root-a1.example.internal
          - host: root-a2.example.internal
          - host: root-a3.example.internal
      - name: us-central1-b
        location: us-central1
        replica_type: READ_WRITE
        root_server:
          - host: root-b1.example.internal
          - host: root-b2.example.internal
          - host: root-b3.example.internal
      - name: us-central1-c
        location: us-central1
        replica_type: READ_WRITE
        root_server:
          - host: root-c1.example.internal
          - host: root-c2.example.internal
          - host: root-c3.example.internal

### Multi-location deployment with witness replica

The following configuration creates a multi-location deployment spanning two data centers and a witness site, with preferred leader placement. The `location_distance` list specifies realistic, asymmetric network latencies between each pair of locations. The two `READ_WRITE` zones each use three root servers for high availability, while the `WITNESS` zone uses `single_server: true` with a single root server because witness replicas don't store user data:

    name: multi-site-deployment
    location:
      - name: datacenter-east
      - name: datacenter-west
      - name: datacenter-central
    location_distance:
      - src: datacenter-east
        dest: datacenter-central
        latency_ms: 25
      - src: datacenter-central
        dest: datacenter-east
        latency_ms: 27
      - src: datacenter-central
        dest: datacenter-west
        latency_ms: 30
      - src: datacenter-west
        dest: datacenter-central
        latency_ms: 32
      - src: datacenter-east
        dest: datacenter-west
        latency_ms: 55
      - src: datacenter-west
        dest: datacenter-east
        latency_ms: 58
    zone:
      - name: east-zone-1
        location: datacenter-east
        replica_type: READ_WRITE
        root_server:
          - host: east-root-1.example.internal
          - host: east-root-2.example.internal
          - host: east-root-3.example.internal
      - name: west-zone-1
        location: datacenter-west
        replica_type: READ_WRITE
        root_server:
          - host: west-root-1.example.internal
          - host: west-root-2.example.internal
          - host: west-root-3.example.internal
      - name: central-witness-zone
        location: datacenter-central
        single_server: true
        replica_type: WITNESS
        root_server:
          - host: witness-root-1.example.internal
    deployment_settings:
      preferred_leader_location: datacenter-east

### Secure deployment with TLS and authentication

The following configuration defines a deployment with mTLS and password authentication enabled:

    name: secure-deployment
    location:
      - name: us-central1
    zone:
      - name: us-central1-a
        location: us-central1
        replica_type: READ_WRITE
        root_server:
          - host: server-1.example.internal
            port_base: 15000
          - host: server-2.example.internal
            port_base: 15000
          - host: server-3.example.internal
            port_base: 15000
    deployment_settings:
      security_settings:
        insecure_mode: false
        authentication_methods:
          - AUTHENTICATION_METHOD_PASSWORD
          - AUTHENTICATION_METHOD_CLIENT_CERTIFICATE
        password_authentication_protocol: PASSWORD_AUTHENTICATION_PROTOCOL_OPAQUE

## What's next

  - [Create a deployment on VMs](https://docs.cloud.google.com/spanner-omni/deploy-on-vms)
  - [Create a deployment with TLS encryption on VMs](https://docs.cloud.google.com/spanner-omni/deploy-encryption-vms)
  - [Scale a VM deployment](https://docs.cloud.google.com/spanner-omni/scale-vm-deployment)
  - [Maintain a deployment](https://docs.cloud.google.com/spanner-omni/maintain-deployment)
