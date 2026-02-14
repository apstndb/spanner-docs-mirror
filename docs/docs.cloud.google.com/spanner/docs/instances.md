This page introduces Spanner instances and their two primary characteristics, instance configurations and compute capacity.

## Overview of instances

To use Spanner, you must first create a Spanner *instance* within your Google Cloud project. This instance is an allocation of resources that is used by Spanner databases created in that instance.

Instance creation includes the following important choices:

  - The [*edition*](/spanner/docs/editions-overview) : Spanner's tier-based pricing model that provides different capabilities at different price points.
  - The [*instance configuration*](/spanner/docs/instance-configurations) : defines the geographic placement and replication topology of the instance.
  - The [*compute capacity*](/spanner/docs/compute-capacity) : the amount of server and storage resources that are available to the databases in an instance.

Once an instance is created, you can list, edit, or delete it. Spanner is a fully managed database service which oversees its own underlying tasks and resources, including monitoring and restarting processes when necessary with zero downtime. As there is no need to manually stop or restart a given instance, Spanner does not offer a way to do so.

Spanner offers a 90-day free trial instance that lets you learn and explore Spanner features and capabilities at no cost for 90 days. To learn more, see [Spanner free trial instances overview](/spanner/docs/free-trial-instance) .

## Spanner editions

Spanner offers different editions to support your various business and application needs. To learn more, see the [Spanner editions overview](/spanner/docs/editions-overview) .

## Instance configurations

An instance configuration defines the geographic placement and replication of the databases in that instance. When you create an instance, you must configure it as one of the following:

  - *Regional* : All resources are contained within a single Google Cloud [region](/docs/geography-and-regions) .
  - *Dual-region* : Resources span two regions in a single country.
  - *Multi-region* : Resources span across multiple regions

You make this choice by selecting an instance configuration, which determines where your data is stored for that instance.

To learn more about instance configurations, see [Regional, dual-region, and multi-region configurations](/spanner/docs/instance-configurations) .

## Compute capacity

Compute capacity defines amount of server and storage resources that are available to the databases in an instance. When you create an instance, you specify its compute capacity as a number of *processing units* or as a number of *nodes* , with 1000 processing units being equal to 1 node.

Which measurement unit you use does not matter unless you are creating an instance whose compute capacity is smaller than 1000 processing units (1 node); in this case, you must use processing units to specify the compute capacity of the instance.

To learn more about compute capacity, see [Compute capacity, nodes and processing units](/spanner/docs/compute-capacity) .

## What's next

  - Learn more about [Spanner editions](/spanner/docs/editions-overview) .
  - Learn more about [regional, dual-region, and multi-region instance configurations](/spanner/docs/instance-configurations) .
  - Learn more about [compute capacity, processing units, and nodes](/spanner/docs/compute-capacity) .
  - Learn how to [Create a Spanner instance](/spanner/docs/create-manage-instances#create-instance) .
