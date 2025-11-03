# Migrating from CircleCI to GitHub Actions and OIDC

## Introduction

This documentation serves as the guiding tool for service teams needing to migrate their builds from CircleCI to GitHub Actions workflows and OIDC.

## Background and motivation

<details>
  <summary> Click here to understand the migration from CircleCI to GitHub Actions. </summary>
Redstone platform used CircleCI as its main continuous integration (CI) tool for all service teams. While it was a robust SaaS tool with advanced features, it had additional costs and limitations that accumulated as HCSS grew.

GitHub Actions, incorporated with the HPE contract, offers the same functionality as CircleCI and more. It is well-integrated with GitHub products and has an active open-source community that offers great support. This aids in minimizing the learning curve on GitHub Actions and leverages many reusable artifacts that are well tested by the community.

OIDC allows for keyless Continuous Integration (CI). The OIDC specs, and Github OIDC implementation in particular, provides an identity protocol, in addition to OAuth 2.0 framework, to enable client applications to rely on authentication that is performed by an OpenID Connect Provider (OP) to verify the identity of a user. OIDC offers passwordless authentication between two trusted systems.

**Note:** See [GitHub Actions](https://docs.github.com/en/actions) and [Configuring OpenID Connect in cloud providers](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-cloud-providers) for additional information.

### Benefitting from keyless CI

#### Easing the burden of authentication

In a zero-trust model, all applications/systems always have to authenticate with one another to gain access to each other resources. To do so, each application/system usually has to maintain multiple set of credentials in the form of _username/password_, _secret keys_, _tokens_ and so on.

For example, a typical HCSS CircleCI workflow needs the following credentials:

- `GITHUB_TOKEN` to interact with GitHub
- `AWS_ACCESS_KEY_ID/AWS_SECRET_ACCESS_KEY` key pairs for accessing AWS resources
- `SONARCLOUD_TOKEN` for accessing Sonar Cloud static code analysis
- Application specific credentials

These credentials require constant maintenance to keep them secure and up-to-date. While many of these credentials are provisioned and rotated automatically, the risk of credential leak is always there. This increases the operational cost on platform and service teams alike.

#### Passwordless authentication with GitHub OIDC

We use AWS extensively and CircleCI does not yet provide an AWS-integrated CI/CD solution. [GitHub OIDC](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services), allows GitHub Actions workflows to access resources in AWS without the need to store the AWS credentials as long-lived GitHub secrets.

By leveraging AWS authentication and authorization management, service teams have more granular control over how workflows access resources. With OIDC, AWS issues a short-lived access token that is only valid for a single job, and then automatically expires.

For more information, see [About security hardening with OpenID Connect](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect) and [Configuring OpenID Connect in Amazon Web Services](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services).

We have created our own [GitHub OIDC App](https://github.com/hpe-hcss/github-oidc-app) to provide an easy, uniform, and secure setup across multiple teams leveraging Github OIDC authentication. This custom application automatically enables common GitHub OIDC use cases within GitHub Actions, by provisioning per-repository AWS roles with configurable policy attachments. These roles are managed according to the lifecycle of the repositories and can be controlled by organization owners.

For more information, see [github-oidc-app](https://github.com/hpe-hcss/github-oidc-app/blob/master/README.md).

### Colocating tools

As GitHub Actions and OIDC are integrated into GitHub, having all the necessary tools in one place/vendor simplifies usage. There is no longer a need to navigate out to a third party, nor maintain the credentials for yet another site.

### Sharing Workflows Across Organizations

GitHub enables sharing workflows and actions across organizations. This streamlines the workflow process as they can be reused rather than recreated.

For more information, see [GitHub Actions](https://docs.github.com/en/actions) and [hpe-actions](https://github.com/hpe-actions).

</details>

## Getting started

The steps below can be done using any text editor can work (VSCode, Vi, Eclipse, Goland, Sublime). Modify code in the editor of choice, and commit to GitHub when complete.

### Configuring GitHub OIDC integration

See [GitHub OIDC App Integration](https://platform-docs.greenlake.hpe.com/docs/security/github-oidc-app.md#integrating-oidc) for the steps required to integrate with the GitHub OIDC App.

### Migrating CircleCI workflows to GitHub actions workflows

See [Defining GitHub Actions Workflows](./github-actions-oidc.md#defining-github-actions-workflows) for the steps required to define GitHub Action workflows on your repositories.

### Removing CircleCI remnants

After completing the previous steps:

1. Verify the desired workflows are now in GitHub Actions, then delete the `.circleci/config.yml` file.
1. Delete `Dockerfile.circleci`.
1. Disable any CircleCI "Required status checks" in the branch protection rules for the repo.
1. Manually stop the builds in [CircleCI](https://support.circleci.com/hc/en-us/articles/360025040233-How-to-stop-building-by-manually-removing-the-CircleCI-webhook-and-deploy-key-from-your-Bitbucket-repository).
1. Remove any secrets rotation, if present.
1. Revoke and remove all secrets from CircleCI project environment variables.
   - View/edit CircleCI environment variables here:

     ```text
     https://app.circleci.com/settings/project/github/<github-org>/<github-project>/environment-variables
     ```

1. For each secret, ensure the values (e.g. AWS keys or GitHub tokens, etc.) are revoked and removed from the source where these secrets were generated.
   - For example, the user and/or keys have been revoked in AWS or GitHub (using the above examples) - not just removed from CircleCI.
1. After revoking each secret at the source, remove the associated environment variables in [CircleCI](https://support.circleci.com/hc/en-us/articles/360025040233-How-to-stop-building-by-manually-removing-the-CircleCI-webhook-and-deploy-key-from-your-Bitbucket-repository).

   **Note:** Any `ECR_AWS_*` secrets are automatically managed/rotated by ecr-lcm, and can be deleted after de-configuring CircleCI.

## Transitioning existing acct-Terraform repos to OIDC-based repos

[This script](https://github.com/hpe-hcss/aws-terraform-config-migration) enables you to transition an existing `acct_terraform` repo to an OIDC-based repo.

## Migrating deploy repos

See [Migrating to OIDC-based Kubernetes Deployments](https://github.com/hpe-hcss/terraform-module-namespace-resources#upgrading) for more information.

## Securely using GitHub actions

See [Securely Using GitHub Actions](./github-actions-oidc.md#securely-using-github-actions).
