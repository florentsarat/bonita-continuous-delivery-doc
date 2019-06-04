# How to continuously deploy Living App artifacts according to each situation

This tutorial describes how to achieve Continuous Deployment of Bonita Living Application artifacts using BCD. You can fully automate deployment to production environment if you want.
Wether the deployment is managed by your CIs jobs or your administrator, is up to you.


## Pre-requisites

Deploying to a production environment (or any sensible environment) requires extra care. Before goind through this tutorial, please make sure you have read and understood the concepts described in these pages:
* [Deploy Living App artifacts](livingapp_deploy.md)
* [Build and deploy (Best Practices)](livingapp_build_and_deploy.md)

## Notion of projects

There is no clear notion of multiple projects yet in Bonita, but the philosophy is already there and the approach will become clearer and clearer with each new version.
As of today we could define a project as a set of artifacts that must work together to fulfill a business need. For instance a loan management project could be composed of several Living Applications, composed of several pages, using multiple REST API extensions and Processes.
Another Health Care project coulb be composed of many processses in a single Living Application using many Custom Connectors.
A development team may implement and maintain several projects concurrently. Having a clear organisation of the code and a Continuous Integration environment is a MUST to be successful.

### First project

For their first project, most of the teams develop all the artifacts in a single Studio project (previsouly named repository), i.e Git repository. While this is very convenient because you have everything in a single place, it also brings some inconsistencies when starting the 2nd project.
Having everything in a single project makes it easy to update artifacts from Bonita Studio. Moreover when building on Continuous Integration (CI) servers, there is only one Git repository to clone and build.

### 2nd project and onward

When do you need to create a 2nd project? Well usually it is a matter of business case. If the same team is implementing a Helthcare application and a Loan Management application, it is easy to see that those applications should be having their own Bonita project (e.g. Git repo).
The rule of thumb is: if the ressources do not share the same lifecycle (i.e. should be updated, versioned and deployed concurrently) they should probably be isolated into separate repos.

Let's assume that the 2nd project is intended to be deployed on the same runtime as the first project. It appears that some ressources shall be shared by the two projects, such as the BDM definition, the organisation, the application Layouts and Themes, etc...
It is usually a bad idea to duplicate code, so you should not duplicate artifacts accross several Bonita projects.

We suggest, you have a project (git repo) with common artifacts and a project (git repo) per business project (e.g. Healthcare or Loan Management).

## Deploying on a test environment

While testing, it is a good idea to always (or most of the time) start from an empty environment and re-install everything. That is the best way to ensure that what you are deploying is all you need to get to the desired state, that you are not relying on an external dependency without noticing it.
In the current version of BCD, the deployment default policies are test oriented, meaning that by default existing artifacts will be removed and re-installed. *Note:* In future version of BCD we will probably change to be production friendly first. This would avoid some of the caveats you may face if you want to target a production environment for your deployment.

### Deployment policy recommanded depends on targeted environment


## Deploying on a pre-production or production environment

The pre-production and production environments must be as similar as possible to ensure that tests are relevant.
Having tests successful must give you confidence enough to deploy the exact same binary to production environment. The configuration is the only thing that should differ from pre-production and production environment.
The installation procedure and binaries should be the same.

### What makes Production a specific environment

In a Production environment all the data are critical and cannot be altered or worst be lost. That is why one should take extra care before deploying anything new on this environment.
As a general rule of thumb, everything deployed on Production environment should be earlier tested and validated on a Pre-production environment.

Please ensure to follow the same procedure on your Pre-production environment and validate that the result is conform to your expectations and that your application is fully operational.

### Deployment policy recommanded for production deployment

Among all the **types of artifacts** that can be deployed (see [Deploy Living App artifacts](livingapp_deploy.md)) there are 4 caveats:

* business data model and business data model access controls
* organization
* profiles
* processes

#### Business Data Model and access control
The business data model is neither scoped to a particular project nor to an application, furthermore it should never be updated in a way that causes a breaking change. A breaking shange is a modification in the structure of the BDM that would break the execution of existing processes.
 If you make such a breaking change, your process instances will never be able to recover the technical error. You will have no choice but to cancel all broken process instances then modify your process definition to follow the change and deploy the new version for your users. It is often better to make changes that are compatible with existing processes already in production, to allow same migration and if needed rollback to earlier version.


#### Organization
While it is possible to define an organization from Bonita Studio using an XML file, this should be considered a test organization. This approach has not been designed for defining a production grade Organization definition.
So please this organization file small and for testing purpose.

The Organization file should probably ignored when deploying to a production environment.
See * [LDAP Synchronizer](https://documentation.bonitasoft.com/bonita/${bonitaDocVersion}/ldap-synchronizer) if your organization can be imported from an LDAP directory.

If you have to deploy an Organization from a file to your production environment you should use the policy: IGNORE_DUPLICATES. This way the users already existing in prodcution will not be modified (e.g. if the passord has been renewed it will be kept as it).
As opposed to the default policy that would reset the user info to what is in the file.


#### Profiles
There is currently only one policy for Profiles: REPLACE_DUPLICATES.
This policy is fine for tests environment as it will reset profiles information to what is in the file.
But when deploying to a production environment, if you had already installed and configure the profile, it will be replace the profile definition by the information in the file, hence you will lose the extra configuration you may have done (e.g. assign profile to groups of users).
As a result, users may lose their access to Living Applications until you restore the configuration.

So it is recommanded not to deploy continuously the profiles definition. As part of your deployment make sure the profiles are installed the first time, then to not deploy them later on, or plan to reconfigure them.

#### Processes
The default policy is deleting existing processes and all their instances. Which is probably not suitable for production environment.

We recommand to use the IGNORE_DUPLICATES policy as it only deploys a process when it does not already exist (same name and version)

### Define the deployment policy for each artifacts

The deployment entry point is called an **Application Archive**. It consists of all artifacts to be deployed and an optional configuration file called **Deployment Descriptor**. This file describes which **Policy** should be applied while deploying each artifact.

The easiest way to configure the deployment is to have several deply.json file, one per targeted environment. Example deploy_test.json, deploy_initial_prod.json and deploy_update_prod.json.

Once the project is built, unzip the Application archive and copy/move/rename the desired deployment descriptor (e.g. deploy_test.json) to deploy.json at the root of the extracted archive.


### Example of an Application structure after extraction

```
bonita-vacation-management-example
├── applications
│   └── Application_Data.xml
├── bdm
│   └── bdm.zip
├── deploy.json
├── extensions
│   └── tahitiRestApiExtension-1.0.0.zip
├── organizations
│   └── ACME.xml
├── pages
│   └── page_ExampleVacationManagement.zip
├── processes
│   ├── Cancel Vacation Request--1.4.1.bar
│   ├── Initiate Vacation Available--1.4.1.bar
│   ├── Modify Pending Vacation Request--1.4.1.bar
│   ├── New Vacation Request--1.4.1.bar
│   └── Remove All Business Data--1.4.1.bar
└── profiles
    └── default_profile.xml
```

See the descriptor at the root of the folder is named *deploy.json*.

### Deploy.json file example for test
```json
{
  "organization": {
    "file": "organizations/ACME.xml",
    "policy": "MERGE_DUPLICATES"
  },
  "profiles": [
    {
      "file": "profiles/default_profile.xml",
      "policy": "REPLACE_DUPLICATES"
    },
    {
      "file": "profiles/custom_profile.xml",
      "policy": "REPLACE_DUPLICATES"
    }
  ],
  "processes": [
    {
      "file": "processes/New Vacation Request--1.4.1.bar",
      "policy": "REPLACE_DUPLICATES"
    },
    {
      "file": "processes/Initiate Vacation Available--1.4.1.bar",
      "policy": "REPLACE_DUPLICATES"
    }
  ],
  "restAPIExtensions": [
    {
      "file": "extensions/tahitiRestApiExtension-1.0.0.zip",
      "policy": "REPLACE_DUPLICATES"
    }
  ],
  "pages": [
    {
      "file": "pages/page_ExampleVacationManagement.zip",
      "policy": "REPLACE_DUPLICATES"
    }
  ],
  "layouts": [
    {
      "file": "layouts/customLayout1.zip",
      "policy": "REPLACE_DUPLICATES"
    },
    {
      "file": "layouts/customLayout2.zip",
      "policy": "REPLACE_DUPLICATES"
    }
  ],
  "themes": [
      {
        "file": "themes/customTheme1.zip",
        "policy": "REPLACE_DUPLICATES"
      },
      {
        "file": "themes/customTheme2.zip",
        "policy": "REPLACE_DUPLICATES"
      }
    ],
  "applications": [
    {
      "file": "applications/Application_Data.xml",
      "policy": "REPLACE_DUPLICATES"
    }
  ],
  "businessDataModel": {
    "file": "bdm/bdm.zip",
    "policy": "REPLACE_DUPLICATES"
  },
  "bdmAccessControl": {
    "file": "bdm/bdm-access-control.xml",
    "policy": "REPLACE_DUPLICATES"
  }
}
```

In this example we decided to deploy a test organization, overwrite profiles and processes.

### Deploy.json file example for production

```json
{
  "processes": [
    {
      "file": "processes/New Vacation Request--1.4.1.bar",
      "policy": "IGNORE_DUPLICATES"
    },
    {
      "file": "processes/Initiate Vacation Available--1.4.1.bar",
      "policy": "IGNORE_DUPLICATES"
    }
  ],
  "restAPIExtensions": [
    {
      "file": "extensions/tahitiRestApiExtension-1.0.0.zip",
    }
  ],
  "pages": [
    {
      "file": "pages/page_ExampleVacationManagement.zip"
    }
  ],
  "layouts": [
    {
      "file": "layouts/customLayout1.zip"
    },
    {
      "file": "layouts/customLayout2.zip"
    }
  ],
  "themes": [
      {
        "file": "themes/customTheme1.zip"
      },
      {
        "file": "themes/customTheme2.zip"
      }
    ],
  "applications": [
    {
      "file": "applications/Application_Data.xml",
      "policy": "REPLACE_DUPLICATES"
    }
  ],
  "businessDataModel": {
    "file": "bdm/bdm.zip"
  },
  "bdmAccessControl": {
    "file": "bdm/bdm-access-control.xml"
  }
}
```

As recommended for production here we neither deploy the organization file nor the profiles (as we know they are already deployed and we want to keep the data available in production).
Processes will not be deleted and re-installed, thanks to the IGNORE_DUPLICATES policy that will only install new processes and let existing ones untouched.


### How to use

Use the `bcd livingapp deploy` command to deploy Living App artifacts:
```
bcd -s <scenario> livingapp deploy -p <path>
```
where:
* **\<scenario>** is the path to the BCD scenario which defines the target Bonita stack. Artifacts will be deployed using tenant credentials defined by this scenario (`bonita_tenant_login` and `bonita_tenant_password` variables).
* **\<path>** is the path to the Application Archive to deploy (directory containing the deploy.json).

You can add a **--debug** option to enable debug mode and increase verbosity.

::: info
Refer to the [BCD Command-line reference](bcd_cli.md) for a complete list of available options for the `bcd livingapp deploy` command.
:::