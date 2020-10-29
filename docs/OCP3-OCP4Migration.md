# Migrating your Application from Openshift 3.x to Openshift 4

## Synopsis

  Technology changes within a blink of an eye. It is important to keep upto date with latest technology to ensure greater efficiency and performance of your applications.
  
  This document helps you with the migration of your application from Openshift 3.11 to Openshift 4. The platform services team have set up two different Openshift Clusters, the on-prem or Silver Cluster and one on Azure, also known as ARO(Azure RedHat Openshift).

  Before migrating your application, it is important to understand the differences between Openshift 3 and Openshift 4. Learn about the differences by clicking [here](https://docs.openshift.com/container-platform/4.2/migration/migrating_3_4/planning-migration-3-to-4.html#migration-comparing-ocp-3-4).

## Disclaimer

  This document has been created based on the migration of DevHub from Openshift 3.11 to Openshift 4. The effort and time mentioned is based on this migration and will remain similar for a full stack web application which has a frontend, backend and a database, if the application has additional components, it will require additional effort. This migration is also specific to an application already present in Openshift 3.11 cluster and this document will be refreshed with time to reflect migrations for other categories of applications.

## Casting and Crew

  It is ideal to have a DevOps Specialist or a maximum of 2 Full Stack Developers in your team for performing this migration. Ensure your team is up to speed with managing CI/CD pipelines and are well versed with the ['oc' command line](https://docs.openshift.com/enterprise/3.0/cli_reference/get_started_cli.html).

## Runtime

  The migration would typically take one sprint(approximately 2 weeks) worth of time to plan and execute the migration.

## Plot

### Assumptions:

 - Your 3-tier full stack web applicaion is running on Openshift 3.11 cluster
 - You have a CI/CD pipeline which covers all stages from building the application to deploying it completely on Openshift
 - You have codified all components or Openshift objects.

### Pre-migration Activities

 -  To test you have all your infrastructure as code, it is recommended to wipe out your development namespace and try deploying to it with zero manual configuration. This includes:

   [] network security policies - sample network security policy files can be found here.
   [] build objects, namely build configs and images
   [] secrets and config maps
   [] application services
   [] state/data
   [] persistent storage
   [] deployment configs
   [] routes

  - Ensure that your CI/CD orchestrator (Jenkins, Circle CI, GitHub Actions, etc) are persistent and stored as code in order to ensure you dont lose the job configurations

  - Ensure that if you have stateful components, the state is synchronized and you have performed a backup. 

  - Ensure that if you have persistent storage volumes, you have backed up the data

  - Identify external dependencies such as:

     [] vanity DNS
     [] external supporting services (SSO, external DB, etc)
     [] network access rules (internal to cluster, external to cluster)
     [] users (direct to user, external applications, internal services)

## Climax 

   You will need to perform the following activities to migrate from OCP 3 to OCP 4

### Network Ingress:
  
   Do you manage an application vanity url? A vanity url is normally of the form (my-app.gov.bc.ca) or do you leverage the pathfinder.gov.bc.ca wildcard? 
   
   With the move to an enterprise service, the platform wildcard has been deemed unsuitable for production application deployments. For exposing tools, dev and test services, the wildcard ingress will still be available at *.apps.silver.devops.gov.bc.ca. This means if you don't have a vanity URL for your application yet, you will want to get started on provisioning one. The steps for getting a url are given below:

   #### Wildcard certs
   
   If you need a wildcart certs, you will need to:
    1) accept risks,
    2) provide valid justifications
    3) dwell with care while using the certificate. 
    
    It would be ideal to engage your MISO before going down that road.

   #### Regular Vanity URLs

   If your require regular vanity urls:

   1) If external facing, get approval for your new URL from your GCPE contact.
   2) Create an iStore request to order certificate (ie. mydomain.gov.bc.ca)
   3) Create Certificate Signing Request(CSR) when order is completed (you will be recieving an email)
   4) Send CSR (keeping private key secure) as instructed, and then you will recieve the cert(s)
   5) Send a DNS request to point new domain url to openshift IP

   For more information on ssl certs abd vanity urls, [click here](https://ssbc-client.gov.bc.ca/services/SSLCert/documents.htm)

### Network Egress

 Many applications also have dependencies on external (out of cluster) services. If access to these services is restricted, external access rules may be required (ie: firewall rules to allow traffic from the silver cluster)
 
 The new silver (and eventually gold) platforms will use a shared egress IP (or range), and restriction to an individual application by IP address will not be supported. (this mimics the current pathfinder network model).
 
 Extending secure communication from applications within the cluster to applications in either another cluster, or the existing datacentre networks is currently under development.

### Internal namespace

Zero trust network security (inside the cluster) will require NetworkSecurityPolicy objects to be present in projects and namespaces to allow traffic to flow between Pods/deployments in a namespace.

With the starting stance of Zero Trust, there will no longer be restrictions on inter-namespace communication either. Both namespaces will need corresponding NetworkSecurityPolicy rules to allow traffic between namespaces.

### Storage Considerations

If your application leverages persistent storage, there are a few differences you can expect in the new Silver platform environment.

The key storageClass types will be available (Block/File). StorageClass names are changing slightly (to better match the storage provisioning.) You may need to modify your manifests to ensure you're using the new storageClasses names where applicable.

   - netapp-block-standard
   - netapp-file-standard

### Enterprise backup integration

A big improvement for integration into the enterprise backup system can be found via a normal persistent volume request for a new storageClass: netapp-file-backup (instead of the service catalog provisioning).

### Migrating data

Existing persistent data will need to be copied from one cluster to another (Sample soon to come to [https://github.com/bcdevops/StorageMigration] repository).

### Build (Docker) considerations

OpenShift 4 now uses podman/buildah instead of docker to build images. You may encounter differences and irregularities with more complex docker builds. The following are a couple of great resources for getting started with podman/buildah.

https://www.openshift.com/blog/openshift-4-image-builds
https://developers.redhat.com/blog/2019/02/21/podman-and-buildah-for-docker-users
https://buildah.io
https://developers.redhat.com/blog/2019/08/14/best-practices-for-running-buildah-in-a-container/

### Time to Migrate

- Migrate your CI/CD orchestrator first. In case you are using Jenkins, migrate Jenkins or redeploy Jenkins to the new tools namespace. This is where a Jenkins stored as code would come in handy. If you are using the BC Gov Jenkins image, use the one [here](https://github.com/BCDevOps/openshift-components/tree/jenkins-basic/upgrade-oc4). If you are using the RedHat Jenkins, use the latest image.

- Modify your CI/CD pipeline to point to the new namespaces and deploy your application.

#### Pros of following this approach

- validates your build and deployment pipeline automation 
- ensures faster return to normalcy 
- identifies potential automation opportunities for the deployment pipeline
- leverages existing integration tests for environment validation
- with a working build/deploy pipeline, only state data is required to be moved to the new cluster
- reduces potential issue surface
- reduces potential cut-over downtime

#### Cons of following this approach

 - requires a completely automated end-to-end CI and CD pipeline
 - you will be required to have a separate mechanism to move over state


## End Credits

https://github.com/BCDevOps/OpenShift4-Migration/tree/master/docs


  