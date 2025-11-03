# Add On Druid Terraform Module

A CCP GitOps Terraform Module for Apache Druid

- [Source repo](https://github.com/glcp/ccp-add-on-druid)
- [Up-to-date documentation](https://platform-devs.greenlake.hpe.com/docs/component-reference/ccp-add-on-druid/README.html)

## Configuration
- [Module Spec](https://github.com/glcp/ccp-add-on-druid/blob/main/resources/MODSPEC.md)
  - [Inputs](https://github.com/glcp/ccp-add-on-druid/blob/main/resources/MODSPEC.md#inputs)
  - [Outputs](https://github.com/glcp/ccp-add-on-druid/blob/main/resources/MODSPEC.md#outputs)
  - [Sub-modules](https://github.com/glcp/ccp-add-on-druid/blob/main/resources/MODSPEC.md#modules)
  - [Resources](https://github.com/glcp/ccp-add-on-druid/blob/main/resources/MODSPEC.md#resources)
  - [Providers](https://github.com/glcp/ccp-add-on-druid/blob/main/resources/MODSPEC.md#providers)
  - [Requirements](https://github.com/glcp/ccp-add-on-druid/blob/main/resources/MODSPEC.md#requirements)

## Usage
Apache Druid is a real-time analytics database designed for analytics ("OLAP" queries) on large data sets.
For more information, see (https://druid.apache.org/docs/latest/design/).

## Releases
- [Latest release](https://github.com/glcp/ccp-add-on-druid/releases/latest)
- [All releases](https://github.com/glcp/ccp-add-on-druid/releases)

## Owners/contacts
- [odavidson@hpe.com](mailto:odavidson@hpe.com)
- [nallagatla@hpe.com](mailto:nallagatla@hpe.com)

## Design/Architecture

### Understanding Druid
Druid is actually 6 separate components managed together, but configured separately. All 6 components require configuring and enabling. 
The components are:
- Broker
- Coordinator
- Historical
- Middlemanager
- Overlord
- Router

The historical component is the only stateful component; it uses local storage to store segment data for fast access.

### Verifying prerequisites
Druid requires the following, created using the Druid Terraform module, aka Druid infra:
- Amazon Aurora PostgreSQL RDS
- Amazon MSK cluster (where metrics are emitted)
- IAM authorization

Also required, are the core infra outputs from the account repo associated with the repo on which you are deploying.

### Terraform provisioning
1. Access the `resources` directory of the desired cluster repository.
1. Create a pull request adding the new desired service as a `.tf`, similar to below:
   ```
   module "druid" {
    source = "git::https://github.com/glcp/ccp-add-on-druid//resources?ref=v0.1.4"

    product_name = "glcs"

    account_id   = local.account_id
    cluster_name = local.cluster_name
    #oidc_idp_arn    = local.oidc_idp_arn
    #oidc_idp_issuer = local.oidc_idp_issuer

    region = local.region

    storage_asg        = local.eks_sg_id
    storage_subnet_ids = local.storage_subnet_ids

    vpc_id   = local.vpc_id
    vpc_cidr = local.vpc_cidr

    iam_auth_enabled = true
    iam_auth_region  = "us-west-2"
    iam_auth_role    = "arn:aws:iam::238491587191:role/oidc/druid-sa-role-validation-glcs-default-eks"

    cloudops_registry_secret = var.cloudops_registry_secret

    global_msk_kafka_connection_string = local.msk_bootstrap_brokers
    postgres_url                       = local.postgres_url
    indexer_bucket_name                = local.bucket_name_indexer
    storage_bucket_name                = local.bucket_name_storage
   }

   ```
1. Platform then approves and merges the PR.
1. After the merge of the new service `.tf`, results in a passing check for `Terraform Plan/Apply/Terraform Apply`.
   - This confirms deployment of the new service to the cluster.

## Deployment
- [Terraform module usage](https://github.com/glcp/ccp-add-on-druid/blob/main/resources/MODSPEC.md)
- A Vault secret containing the password for the Postgres endpoint MUST be created manually before deployment.
This secret should be located at `ccp/glcs/infra/data/aurorapostgres/default`.

```terraform

module "module_name" {
  source = "github.com/glcp/ccp-terraform-aws-module.git//resources?ref=\<VERSION>"
}

```

## Upgrading/Downgrading
- Not supported

## Automated Testing
- Not applicable

### Verify plan
- Not applicable

### Verify apply
- Not applicable

## Manual Testing
1. Make any desired changes to the helm chart.
1. Create a PR for [Kleo](mailto:odavidson@hpe.com) and [Teja](mailto:nallagatla@hpe.com) to review.
1. Kleo and Teja test the changes on a test cluster.

## Troubleshooting
- When having issues, verify the following:
  - Are the pods running and healthy?
    - Are all 6 subcomponents functioning?
  - Port forward into Druid UI, verify any and all data is correctly reflected.
- Because of the local storage strategy, nodes on the cluster require correct labeling for the historical subcomponent to run correctly.
- If data is streaming but, cuts off at a certain point, this means the connection to Amazon RDS is misconfigured.
  - Generally the db table name is incorrect, but there are other potential issues.
- Occasionally, the connection to S3 and RDS are broken by missing or incorrect credentials in Vault.

## Contributing
- [How to contribute](https://github.com/glcp/ccp-add-on-druid/blob/main/CONTRIBUTING.md)

