# Migrating your Application from Openshift 3.x to Openshift 4

## Synopsis

  Technology changes within a blink of an eye. It is important to keep upto date with latest technology to ensure greater efficiency and performance of your applications.
  
  This document helps you with the migration of your application from Openshift 3.11 to Openshift 4. The platform services team have set up two different Openshift Clusters, the on-prem or Silver Cluster and one on Azure, also known as ARO(Azure RedHat Openshift).

  Before migrating your application, it is important to understand the differences between Openshift 3 and Openshift 4. Learn about the differences by clicking [here](https://docs.openshift.com/container-platform/4.2/migration/migrating_3_4/planning-migration-3-to-4.html#migration-comparing-ocp-3-4).

## Disclaimer

  This document has been created based on the migration of DevHub from Openshift 3.11 to Openshift 4. The effort and time mentioned is based on this migration and will remain similar for a full stack web application which has a frontend, backend and a database, if the application has additional components, it will require additional effort. This migration is also specific to an application already present in Openshift 3.11 cluster and this document will be refreshed with time to reflect migrations for other categories of applications.

## Casting and Crew

  It is ideal to have a DevOps Specialist or a maximum of 2 Full Stack Developers in your team for performing this migration.

## Runtime

  The migration would typically take one sprint(approximately 2 weeks) worth of time to plan and execute the migration.

## Plot

### Assumptions:

 - Your 3-tier full stack web applicaion is running on Openshift 3.11 cluster
 - You have a CI/CD pipeline which covers all stages from building the application to deploying it completely on Openshift
 - You have codified all components or Openshift objects.

### Pre-migration Activities

 -  To test you have all your infrastructure as code, it is recommended to wipe out your development namespace and try deploying to it with zero manual configuration. This includes:

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

  - Identify external dependencies