---
title: MTA Analysis
date: "2024-02-29"
---

# MTA - migration analysis workflow

# Synopsis

This workflow is an assessment workflow type, that invokes an application analysis workflow using [MTA][1]
and returns the [move2kube][3] workflow reference, to run next if the analysis is considered to be successful.

Users are encouraged to use this workflow as self-service alternative for interacting with the MTA UI. Instead of running
a mass-migration of project from a managed place, the project stakeholders can use this (or automation) to regularly check
the cloud-readiness compatibility of their code.

# Inputs

- `repositoryUrl` [mandatory] - the git repo url to examine
- `recipients` [mandatory] - A list of recipients for the notification in the format of `user:<namespace>/<username>` or `group:<namespace>/<groupname>`, i.e. `user:default/jsmith`.

# Output

1. On completion the workflow returns an [options structure][2] in the exit state of the workflow (also named variables in SonataFlow)
   linking to the [move2kube][3] workflow that will generate k8s manifests for container deployment.
1. When the workflow completes there should be a report link on the exit state of the workflow (also named variables in SonataFlow)
   Currently this is working with MTA version 6.2.x and in the future 7.x version the report link will be removed or will be made
   optional. Instead of an html report the workflow will use a machine friendly json file.

# Dependencies

- MTA version 6.2.x or Konveyor 0.2.x

  - For OpenShift install MTA using the OperatorHub, search for MTA. Documentation is [here][1]
  - For Kubernetes install Konveyor with olm
    ```bash
    kubectl create -f https://operatorhub.io/install/konveyor-0.2/konveyor-operator.yaml
    ```

# Runtime configuration

| key                                                  | default                                                                                    | description                               |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------------ | ----------------------------------------- |
| mta.url                                              | http://mta-ui.openshift-mta.svc.cluster.local:8080                                         | Endpoint (with protocol and port) for MTA |
| quarkus.rest-client.mta_json.url                     | ${mta.url}/hub                                                                             | MTA hub api                               |
| quarkus.rest-client.notifications.url                | ${BACKSTAGE_NOTIFICATIONS_URL:http://backstage-backstage.rhdh-operator/api/notifications/} | Backstage notification url                |
| quarkus.rest-client.mta_json.auth.basicAuth.username | username                                                                                   | Username for the MTA api                  |
| quarkus.rest-client.mta_json.auth.basicAuth.password | password                                                                                   | Password for the MTA api                  |

All the configuration items are on [./application.properties]

# Testing

User can test the MTA workflow by providing a git repository.

#### Existing Repository

An existing repository can be used. Alternatively, user can use example repo: [spring-petclinic][7]

#### New Repository

- Create a git repo with a simple Java class. It can either be public or private.
  Currently, MTA supports mostly Java projects.
- If repository is private, refer to [configuring repository][4] on how to set up credentials on MTA.
  - Note: When creating the source control credentials in MTA (usually done by MTA admin), ensure to input personal access token in the password field.
    Refer to [personal access token] [5] on how to create personal access token (classic) on GitHub.
  - Also, git url address must use https. Refer to [about remote repositories][6] for more details.
- If the repo is public, no further configuration is needed.

# Workflow Diagram

![mta workflow diagram](https://github.com/parodos-dev/serverless-workflows/blob/main/mta/mta.svg?raw=true)

[1]: https://developers.redhat.com/products/mta/download
[2]: https://github.com/parodos-dev/serverless-workflows/blob/main/assessment/schema/workflow-options-output-schema.json
[3]: https://github.com/parodos-dev/serverless-workflows/tree/main/move2kube
[4]: https://docs.redhat.com/en/documentation/migration_toolkit_for_applications/6.2/html-single/user_interface_guide/index#configuring-credentials
[5]: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic
[6]: https://docs.github.com/en/get-started/getting-started-with-git/about-remote-repositories
[7]: https://github.com/spring-projects/spring-petclinic
