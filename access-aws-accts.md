# Accessing SSO Accounts

**Note**: FinOps monitors or restricts usage to the AWS Marketplace as some offerings can be costly or outside budget controls. Therefore, a Service Control Policy (SCP) is in place regarding enabling a model in AWS Bedrock.

## Accessing AWS accounts
To access an AWS account, perform the following steps:
1. Log into the SSO portal for your [organization](#team-links)
   - A list of all AWS accounts with access opens.
     **Note**:  If unable to access the desired team link, make sure you have been added to IdC (and appropriate groups, as needed). See [Working with the IAM Identity Center](https://platform-docs.greenlake.hpe.com/docs/accts-subscrips/aws/aws-user-accts/access-aws-resources/#working-with-the-iam-identity-center) for details.
1. You can also search for an AWS account with an account name or AWS account ID
1. To see the management console, click **Management console** next to the desired account. 
1. To see the CL/Programmatic access, click **Command line and programmatic access**. 
1. If selecting **Command line and programmatic access**, the Get Credentials for... page opens. Select the needed OS. 
   - Export commands change per OS. This simplifies assignments. 
   
### Creating a CLI profile

The [AWS Command Line Interface (AWS CLI)](https://aws.amazon.com/cli/) is a unified tool to manage your AWS services.

To create the profile for CLI, see [Using an IAM Identity Center named profile](https://docs.aws.amazon.com/cli/latest/userguide/sso-using-profile.html) for more information.

### Configuring AWS CLI to use AWS IdC

To configure the AWS CLI to use AWS IdC, see [Token provider configuration with automatic authentication refresh for AWS IAM Identity Center (successor to AWS Single Sign-On)](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html) for more information.

Below are the values to use when running the `aws sso configure` command:

- Use the SSO link for your organization listed in the [table below](#team-links)
- "SSO registration scopes": `sso:account:access`

## Access requests for CCP Engineers

CCP engineers' access to AWS accounts and clusters is managed in the `ou_groups` directory under `inputs`. 

### Specific files
- **QA Accounts and Clusters**: Managed in `ou_ccp_qa.yaml`.
- **Production Accounts and Clusters**: Managed in `ou_ccp_prod.yaml`.

Access is managed at the **domain group** level. To request access, add the user to the respective domain group. 
For on-call duties, also assign the user to the `ccp_oncall` group.

**Note**:
- Two scheduled workflow runs `on-call-engineer_rotation_add_users.yml` and `on-call-engineer_rotation_remove_users.yml` keep the on-call user list in sync with pagerduty

- When requesting access, ensure the account number is present in the [account list](https://github.com/hpe-sre/aws-ccp_idc-config/blob/main/inputs/ou_groups/ou_ccp_qa.yaml#L218). If it is not, please add the account number to the list.

## Access requests for BU Engineers

BU engineers' access is managed under the `ou_accounts` directory.

- Most BU access is managed in `ou_root.yaml`.
- Storage-specific access is managed in `ou_storage.yaml`.

### Planned updates
We are planning to break down `ou_root.yaml` into dedicated BU groups, such as:
- `ou_compute.yaml`
- `ou_ccs.yaml`
- `ou_melody.yaml`

### Instructions
1. Search for the requested account number.
1. Locate the associated permission set.
1. Add the user to the user group that corresponds to the requested role access.

### Pull Request (PR) titles and description

When submitting a PR, ensure the title is descriptive for the reviewers. Include:
- Organization name
- Environment
- Role for which access is requested
- Account name
- Why (if write access requested)

### Example
`Melody(QA): Add Jon to melody_ops_ro in arcus-qa account`

### Correcting an IdC loop
If you are in an SSO IdC loop:
1. Open a new tab and navigate to: https://login-itg-iam.ext.hpe.com/idp/SSO.saml2
1. Access debug tools (F12) > **Application** > **Storage**.
1. Click **Clear site data**.
1. Open a new tab and navigate to: https://d-92670f37c5.awsapps.com/
1. Access debug tools (F12) > **Application** > **Storage**.
1. Click **Clear site data**.

Login should work in the original tab. You need to re-enter your credentials.

## Updating user access for accounts
You need to update the organizational unit (OU) account file to grant or remove account access. 
1. Find the IAM IdC user access list for an account by searching for the account number or name in the input files located in the OU Accounts directory listed [below](#team-links)
   - Input files are organized by AWS organizational units.
1. Create a PR adding or removing users to the appropriate groups as necessary.
   - Forking the repository may be necessary
   - Users lists must be kept in alphabetical order 
1. Post the PR in the [#pie-engineering-partners](https://hpe.enterprise.slack.com/archives/C05G08K3C0H) Slack channel to get approvals.
   **Note:** Permissions cascade down in default groups without needing to include their name in each list:
   - Users in `admin` receive the `poweruser` and `readonly` roles.
   - Users in `poweruser` receive the `readonly` role.
   - Users in `k8s_rw` receive the `k8s_ro` role.

### Working with Unified ID
To obtain the necessary signing certificate from IT, see [Unified ID FAQ](https://hpe.sharepoint.com/teams/unifiedid) to file a ServiceNow ticket.
An EPRID is required for production certificates. Without an EPRID a POC certificate is possible to request and use temporarily.
EPRID's are stored in CMDB (available from https://hpe.service-now.com/).

## Team Links

| **Org Name** | **AWS SSO Login** | **IdC Repo** | **OU Accounts Directory** |
|---|---|---|---|
| GLCS | <https://hpe-glc.awsapps.com/start> | <https://github.com/hpe-sre/aws-glcs_idc-config/tree/main> | <https://github.com/hpe-sre/aws-glcs_idc-config/tree/main/inputs/ou_accounts> |
| CCP | <https://ccp.awsapps.com/start> | <https://github.com/hpe-sre/aws-ccp_idc-config> | <https://github.com/hpe-sre/aws-ccp_idc-config/tree/main/inputs/ou_accounts> |
| Aruba | <https://aruba.awsapps.com/start/> | <https://github.com/hpe-sre/aws-aruba_idc-config> | <https://github.com/hpe-sre/aws-aruba_idc-config/tree/main/inputs/ou_accounts> |
| Reference Architecture |  | <https://github.com/hpe-sre/aws-ra_idc-config> | <https://github.com/hpe-sre/aws-ra_idc-config/tree/main/inputs/ou_accounts> |

## Account Create Repository Link
<https://github.com/hpe-sre/account-create-requests>    
