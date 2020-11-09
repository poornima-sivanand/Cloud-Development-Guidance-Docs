# Migrating your Application from Openshift 3.x to Openshift 4

## Introduction

  Technology changes within a blink of an eye. It is important to keep upto date with latest technology to ensure greater efficiency and performance of your applications.
  
  This document helps you with the migration of your application from Openshift 3.11 to Openshift 4. The platform services team have set up two different Openshift Clusters, the on-prem or Silver Cluster and one on Azure, also known as ARO(Azure RedHat Openshift).

  Before migrating your application, it is important to understand the differences between Openshift 3 and Openshift 4. Learn about the differences by clicking [here](https://docs.openshift.com/container-platform/4.2/migration/migrating_3_4/planning-migration-3-to-4.html#migration-comparing-ocp-3-4).

  This document has been created based on the migration of DevHub from Openshift 3.11 to Openshift 4. The effort and time mentioned is based on this migration and will remain similar for a full stack web application which has a frontend, backend and a database, if the application has additional components, it will require additional effort. This migration is also specific to an application already present in Openshift 3.11 cluster and this document will be refreshed with time to reflect migrations for other categories of applications.


## Team Composition 

  It is ideal to have a DevOps Specialist or a maximum of 2 Full Stack Developers in your team for performing this migration. Ensure your team is up to speed with managing CI/CD pipelines and are well versed with the ['oc' command line](https://docs.openshift.com/enterprise/3.0/cli_reference/get_started_cli.html).

## Time required for migration

  The migration of a full stack web application is considered to be of medium complexity. The migration would typically take one sprint(approximately 2 weeks) worth of time to plan and execute the migration.


## Assumptions:

 - Your 3-tier full stack web applicaion is running on Openshift 3.11 cluster
 - You have a CI/CD pipeline which covers all stages from building the application to deploying it completely on Openshift
 - You have codified all components or Openshift objects.

## Pre-migration Activities

 -  To test you have all your infrastructure as code, it is recommended to wipe out your development namespace and try deploying to it with zero manual configuration. This includes:

        [ ] network security policies - sample network security policy files can be found here  

        [ ] build objects, namely build configs and images  

        [ ] secrets and config maps  

        [ ] application services  
        
        [ ] state/data

        [ ] persistent storage  

        [ ] deployment configs   
        
        [ ] routes

  - Ensure that your CI/CD orchestrator (Jenkins, Circle CI, GitHub Actions, etc) are persistent and stored as code in order to ensure you dont lose the job configurations

  - Ensure that if you have stateful components, the state is synchronized and you have performed a backup. 

  - Ensure that if you have persistent storage volumes, you have backed up the data

  - Identify external dependencies such as:

        [ ] vanity DNS  

        [ ] external supporting services (SSO, external DB, etc)

        [ ] network access rules (internal to cluster, external to cluster)

        [ ] users (direct to user, external applications, internal services)


## Migration Considerations

   You will need to perform the following activities to migrate from OCP 3 to OCP 4

### Network Ingress
  
   Do you manage an application vanity url? A vanity url is normally of the form (my-app.gov.bc.ca) or do you leverage the pathfinder.gov.bc.ca wildcard? 
   
   With the move to an enterprise service, the platform wildcard has been deemed unsuitable for production application deployments. For exposing tools, dev and test services, the wildcard ingress will still be available at *.apps.silver.devops.gov.bc.ca. This means if you don't have a vanity URL for your application yet, you will want to get started on provisioning one. The steps for getting a url are given below:

##### Wildcard certs
   
If you need a wildcart certs, you will need to:

    1) accept risks

    2) provide valid justifications

    3) dwell with care while using the certificate. 

    
 It would be ideal to engage your MISO before going down that road.

##### Regular Vanity URLs

 If your require regular vanity urls:

    1) If external facing, get approval for your new URL from your GCPE contact

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

### Working with multiple clusters

When performing a migration from one cluster to another it is important that you can rapidly change between clusters without having to manually login. [Click here](https://developer.gov.bc.ca/Working-in-Multiple-Clusters) to learn more about contexts and how to switch between clusters on the fly.

### Migrating data

Existing persistent data will need to be copied from one cluster to another (Sample soon to come to [https://github.com/bcdevops/StorageMigration] repository).

### Build (Docker) considerations

OpenShift 4 now uses podman/buildah instead of docker to build images. You may encounter differences and irregularities with more complex docker builds. The following are a couple of great resources for getting started with podman/buildah.

https://www.openshift.com/blog/openshift-4-image-builds
https://developers.redhat.com/blog/2019/02/21/podman-and-buildah-for-docker-users
https://buildah.io
https://developers.redhat.com/blog/2019/08/14/best-practices-for-running-buildah-in-a-container/

## Time to Migrate

- Migrate your CI/CD orchestrator first. In case you are using Jenkins, migrate Jenkins or redeploy Jenkins to the new tools namespace. This is where a Jenkins stored as code would come in handy. If you are using the BC Gov Jenkins image, use the one [here](https://github.com/BCDevOps/openshift-components/tree/jenkins-basic/upgrade-oc4). If you are using the RedHat Jenkins, use the latest image.

If you are using bcgov jenkins, [click here](https://developer.gov.bc.ca/Migrating-Your-BC-Gov-Jenkins-to-the-Cloud) to see how to migrate it.

- Modify your CI/CD pipeline to point to the new namespaces and deploy your application.

- If you are migrating rhel images, [click here](https://developer.gov.bc.ca/Migrating-Rhel-Images)to know how to migrate your applications.

[Click here](https://github.com/BCDevOps/OpenShift4-Migration/blob/master/docs/pre-migration-cl.md) to identify different migration possibilities.

## Possible delays

  The possible reasons for a delay in your migration could be because of:

  - DNS migration : DNS migrations could take some time to reflect due to the Time To Live property of DNS records. Time To Live is the expiry time of a DNS record. For e.g., if the TTL is set to 86400 seconds or 24 hrs, it means that once the DNS record has been requested, this request will cached for 86400 seconds before it is re-requested. 

  - Availability/Outage requirements: There might be a need to schedule a downtime for your application before you migrate it to the new cluster, which could lead to potential delays.

  - Phase dependencies/prerequisites: Migrations could possibly be delayed due to the time required to move state or if all of your infrastructure hasnt been maintained as code (such as secrets, config maps etc.)

  [Click here](https://developer.gov.bc.ca/App-Migration-Painpoints) to know possible pain points of migration.


## FAQs

### 1. What is the version of OpenShift 4 available to BC Gov?

**Ans**: Openshift 4.5.7 is the current deployed version - the plan is to be up to date (current release as of starting the early access)

### 2. Will any migration tools be available?

**Ans**: The Cluster Application Migration (CAM) will be available but since it does not currently support multi-tenancy, the migration using this tool will be assisted by a member of the Platform Services Team.  
The recommended path will be to deploy your application pipeline and validate your application through the different environments (dev/test/prod-like), saving the final cut-over of prod until you're satisfied with your deployments. This approach allows you to fully validate your entire application before doing any final state/Service cutover from one cluster to the other.

[Click here](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.2/html/migration/migrating-from-openshift-container-platform-3#migration-understanding-cam_migrating-3-4) to learn more about the Cluster Application Migration tool.

### 3. Are operators available, if so which of them will be initially available? 

**Ans**:  Operators will be available, we have not yet built a list of operators that will be available for use (home built operators for artifactory and aporeto will be available to leverage those services right away.)
- the work with operators that's still outstanding is work to determine the deployment patterns for operators (ie: cluster/platform-services managed and updated, vs individually deployed operators that can be scoped to a narrow set of namespaces, etc.)

Operators require cluster installation, and therefore will need coordination with the platform services team.

Some of the questions the platform services team is looking for answers are:
- does the operator have to be managed by the platform-services team?
- can we have multiple versions of an operator installed?
- can we have different namespace scopes (access) for each version of an operator?

Each operator can have different answers for each of these questions.

### 4. Will we be able to use custom resource definitions (CRDs)?

**Ans**: CRDs are still a cluster configuration. Adding new CRDs will require coordination with the platform services team for their deployment/RBAC/etc.

### 5. Given that the new platform will use Container Runtime Interface-Openshift(CRI-O), and that the [documentation](https://docs.openshift.com/container-platform/4.6/openshift_images/create-images.html) says "CRI-O supports the insertion of random user IDs into the container’s /etc/passwd, so changing it’s permissions should never be required.", does this means that we won't need to have custom entrypoints to write the random uid/gid in /etc/passwd with OCP4?

**Ans**: When OpenShift starts a container, it uses an arbitrarily assigned user ID. This feature helps to ensure that if an application from within a container manages to break out to the host, it won’t be able to interact with other processes and containers owned by other users, in other projects. If the process has requirements to alter file permissions or retrieve user information, then this security feature will cause problems for the container. Typically the advice has been to allow the root group to read/write files and directories, by changing the group ownership of those files to root or allowing the random user information to be added to the /etc/passwd file. 

This led to a known vulnerability by making the /etc/passwd file group readable. The OpenShift run-time CRI-O (starting from OpenShift 4.2 onward) now inserts the random user for the container into /etc/passwd. Removing the requirement to insert the random user manually into /etc/passwd completely. Additionally in future versions of CRI-O, the $HOME or $WORKDIR of the container user will also be assigned, helping Java based images. To read more about this, [click here](https://access.redhat.com/articles/4859371)

### 6. Will Jenkins be the default OCP4 pipeline or should we be prepared to migrate our CI/CD pipelines to a new solution (Tekton, set up our own solution, GH actions etc.)?

**Ans**: Jenkins will be available for migration, but new development teams should use another solution. Tekton will be available in Openshift 4.7. If you are using Jenkins, you will still need to configure your Jenkins deployment in your tools environment which would be a great first step in your migration journey.

### 7. Will there be the same Jenkins auto-provisioning as 3.11 or is that gone?

**Ans**: If you're refering to the tools,```build pipeline``` functionality of OCP3 (that would launch jenkins for you), this feature is not expected to continue. If you are using jenkins pipelines, you'll want to configure a long lived deployment in your tools namespace to ensure it's always watching your repo.

### 8. Is there a public repo in BC Gov GitHub which uses the GitHub Actions pipeline for deployment?

**Ans**:  Checkout https://github.com/bcgov/platform-services-registry/tree/master/.github/workflows for a sample repo that uses Github Actions to deploy the applications to Openshift.

 
### 9. Does anybody plan on bringing in an ocp 4 specific feature to their deployments out of the gate, or are most people planning like-for-like for their migration?

**Ans**: Mostly early access teams are looking towards working on like-for-like migration and are starting off my migrating their CI/CD pipelines, but are interested in extra metrics and tracing that Istio offers, Vault for storing and managing secrets, shifting from Jenkins to Tekton, Mongo 4 and other database operators.

Look for https://www.openshift.com/streaming for Openshift Demos from Openshift Product Managers

###  10. We have some routes like *.pathfinder.gov.bc.ca created on Openshift v3, after pathfinder cluster is gone, what should they look like on Openshift v4?

**Ans**: If you have production services, we strongly recommend that you get a vanity DNS record setup that's NOT tied to the OCP cluster. (eg: dev.oidc.gov.bc.ca instead of sso-dev.pathfinder.gov.bc.ca)

For non-prod (non-published) service names, the silver cluster will provide a wildcard DNS/cert combo at *.apps.silver.devops.gov.bc.ca

### 11. Is something like *.nrs.gov.bc.ca considered a vanity url or should it be *.apps.gov.bc.ca?

**Ans**:  (Need more info on managed DNS)

### 12. Will the Trident Provisioner issue affect the OCP4 cluster?

**Ans**: No. It will not affect the OCP4 cluster. (Need more info the what was the trident provisioner issue)

### 13. What is Artifactory? Where can I find its documentation?

**Ans**: The Developer Experience Team provides Artifactory for two purposes:

1. Caching artifacts from the public internet to allow faster builds.
2. Private repositories for artifacts that teams create/use and which cannot be provided to the public.

You can read more about artifactory [here](https://github.com/BCDevOps/developer-experience/blob/cailey/artifactory/directory-fixes/apps/artifactory/DEVHUB-README.md)

### 14. How do I add a user in OCP4? In the User Management, all I see is Service Accounts, Roles, Role Bindings.

**Ans**: To add a user, switch from Admin Access to Developer Access, select the project and then click on Project Access

### 15. To add users, should we use githubuser@github or githubuser@github.com?

**Ans**: It should be githubuser@github.

### 16. Has anyone here included rolebinding manifests in their project setup yet? (or is everyone still running with the GUI?)

**Ans**: 

### 17. Tried a quick deployment of our Azure Agent and I'm getting a network error "Cannot initiate the connection to archive.ubuntu.com" and lots of different errors like that.

**Ans**: For any network issues, first step is to check if you have enabled the network security policies. Information on network security policies and how to set them up can be found [here](https://developer.gov.bc.ca/NetworkSecurityPolicy:-Quick-Start).

A sample network security policy file to apply to your openshift namespace can be found [here](https://github.com/BCDevOps/platform-services/blob/master/security/aporeto/docs/sample/quickstart-nsp.yaml).

### 18. Will the network security policies be set by default in the namespaces?

**Ans**: No. You will need to set up the network security policy on each of the 4 namespaces (tools, dev, test and prod).

### 19. In Openshift 3, setting role bindings for users and system accounts could easily be done via the Openshift console. That doesnt seem to be the case with OCP4. How can I set tole bindings in OCP4?

**Ans**: It is recommended to store all your RBAC as manifest files in your repository to manage and organize them better. 

An example RBAC manifest file can be found [here](https://github.com/bcgov/platform-services-registry/blob/00cd113ad2e8735e16404bb96c88ba0599dac7eb/openshift/templates/project-set-rbac.yaml).

For More FAQs refer [here](https://github.com/BCDevOps/OpenShift4-Migration/issues)

## References

https://github.com/BCDevOps/OpenShift4-Migration/tree/master/docs


  