# AkashPersistentStorageSDLDeepdive
Testnet 3
Akash Persistent Storage
Overview
Akash persistent storage allows deployment data to persist through the lifetime of a lease.  The provider creates a volume on disk that is mounted into the deployment.  This functionality closely mimics typical container persistent storage.
Persistent Storage Limitations
Please note that storage only persists during the lease.  The storage is lost when:
The deployment is updated as this generates a new lease.
The deployment is migrated to a different provider.
The deployment’s lease is closed.  Even when relaunched onto the same provider, storage will not persist across leases.
Implementation Overview
Configuration Examples
For the purposes of our review this SDL Example will be used.
Backward Compatibility
The introduction of persistent storage does not affect your pre-existing SDLs.  All manifests authored previously will work as is with no change necessary.
Troubleshooting Tips
If any errors occur in your deployments of persistent storage workloads, please review the troubleshooting section for possible tips.  We will continue to build on this section if there are any frequently encountered issues.
Persistent Storage SDL Deepdive
Our review highlights the persistent storage parameters within the larger SDL example.
Resources Section
Overview

Within the profiles > compute > <profile-name> resources section of the SDL storage profiles are defined.  Our review begins with an overview of the section and this is followed by a deep dive into the available parameters.

NOTE - a  maximum amount of two (2) volumes per profile may be defined.  The storage profile could consist of:
A mandatory local container volume which is created with only a size key in our example
An optional persistent storage volume which is created with the persistent: true attribute in our example

profiles:
  compute:
    grafana-profile:
      resources:
        cpu:
          units: 1
        memory:
          size: 1Gi
        storage:
          - size: 512Mi
          - name: data
            size: 1Gi
            attributes:
              persistent: true

Name
Each storage profile has a new and optional field name.  The name is used by services to link service specific storage parameters to the storage. It can be omitted for single value use case and default value is set to default.
Attributes

A storage volume may have the following attributes.

persistent - determines if the volume requires persistence or not. The default value is set to false.

class - storage class for persistent volumes. Default value is set to default. NOTE - It is invalid to set storage class for non-persistent volumes.  Storage volume class types are expanded upon in the subsequent section.

Storage Class Types

The class allows selection of a storage type.  Only providers capable of delivering the storage type will bid on the lease.

The selection of the default class allows the provider to assign their default storage type to the lease.

Class Name
Throughput/Approx matching device	
beta1
hdd
beta2
ssd
beta3
NVMe
default
Provider defined default class


Services Section
Overview
Within the services > <service-name> section a new params section is introduced and is meant to define service specific settings.  Currently only storage related settings are available in params.  Our review begins with an overview of the section and this is followed by a deep dive into the use of storage params.

services:
  postgres:
    image: postgres
    params:
      storage:
        data:
          mount: /var/lib/postgres

Params

Note that params is an optional section under the greater services section.  Additionally note that non-persistent storage should not be defined in the params section.  In this example profile section, two storage profiles are created.  The no name ephemeral storage is not mentioned in the services > params definition.  However the persistent storage profile, named data, is defined within services > params
profiles:
  compute:
    grafana-profile:
      resources:
        cpu:
          units: 1
        memory:
          size: 1Gi
        storage:
          - size: 512Mi
          - name: data
            size: 1Gi
            attributes:
              persistent: true


Storage
The persistent volume is mounted within the container’s local /var/lib/postgres directory.

     params:
      storage:
        default:
          mount: /var/lib/postgres


Alternative Uses of Params Storage

Default Name Use

In this example the params > storage section is defined for a storage profile using the default (no name explicitly defined) profile

Services Section
services:
  postgres:
    image: postgres
    params:
      storage:
        data:
          mount: /var/lib/postgres


Profiles Section
profiles:
  compute:
    grafana-profile:
      resources:
        cpu:
          units: 1
        memory:
          size: 1Gi
        storage:
          - size: 512Mi
          - size: 1Gi
            attributes:
              persistent: true
              class: beta2


Multiple Container Share of a Single Persistent Volume

In the following example two containers mount a single persistent volume.  Both containers would have access to the single volume, named data, via the specified local directory.

Services Section
services:
  postgres:
    image: postgres
    params:
      storage:
        data:
          mount: /var/lib/postgres
  grafana:
    image: grafana/grafana
    params:
      storage:
        data:
          mount: /var/lib/grafana


Profiles Section
profiles:
  compute:
    grafana-profile:
      resources:
        cpu:
          units: 1
        memory:
          size: 1Gi
        storage:
          - size: 512Mi
          - name: data
            size: 1Gi
            attributes:
              persistent: true
              class: beta2
    postgres-profile:
      resources:
        cpu:
          units: 1
        memory:
          size: 1Gi
        storage:
          - size: 512Mi
          - name: data
            size: 10Gi
            attributes:
              persistent: true
              class: beta2

Troubleshooting
Hostname Conflict - May Cause Manifest Send Errors
The example SDL used in this guide and the samples in the GitHub repo contain an accept field that defines the hostname that the deployment will accept inbound connections for.

If the hostname defined in the accept field is already in use within the Akash provider, a conflict occurs if another deployment attempts to launch with the same hostname.  This could occur within our testnet environment if multiple people are attempting to use the same SDL and deploy to the same provider.  Consider changing the accept field to a unique hostname (I.e. <myname>.locahost) if you receive an error in send of the manifest to the provider.
 
 grafana:
    image: grafana/grafana
    expose:
      - port: 3000
        as: 80
        to:
          - global: true
        accept:
          - webdistest.localhost

Complete Persistent Storage Manifest/SDL Example

version: "2.0"
services:
  postgres:
    image: postgres
    params:
      storage:
        data:
          mount: /var/lib/postgres
  grafana:
    image: grafana/grafana
    expose:
      - port: 3000
        as: 80
        to:
          - global: true
        accept:
          - webdistest.localhost
    params:
      storage:
        data:
          mount: /var/lib/grafana
profiles:
  compute:
    grafana-profile:
      resources:
        cpu:
          units: 1
        memory:
          size: 1Gi
        storage:
          - size: 512Mi
          - name: data
            size: 1Gi
            attributes:
              persistent: true
              class: beta2
    postgres-profile:
      resources:
        cpu:
          units: 1
        memory:
          size: 1Gi
        storage:
          - size: 512Mi
          - name: data
            size: 10Gi
            attributes:
              persistent: true
              class: beta2
  placement:
    westcoast:
      attributes:
        region: us-west
      pricing:
        grafana-profile:
          denom: uakt
          amount: 1000
        postgres-profile:
          denom: uakt
          amount: 7000
deployment:
  grafana:
    westcoast:
      profile: grafana-profile
      count: 1
  postgres:
    westcoast:
      profile: postgres-profile
      count: 1



