# About the repository template
This repo serves as a template for establishing a new repository for a project at IMPACT. It follows [Gitflow](https://github.com/NASA-IMPACT/ghgc-data-airflow/blob/dev/GITFLOW.md) principles and establishes a good baseline from which to build your projects. Please be sure to read the following information to make sure your project is set up correctly.

## Creating a repository from this template
- Above the file list, click **Use this template** and select **Create a new repository**.
- From the **Owner** drop-down menu select the account you want to own the new repository
- Name the repo using the format project_team-application in lower case (eg. admg-backend)
- Ensure that **Include all branches** is selected to ensure all branches and directories are copied
- Click **Create repository from template**

## Setting the branch protection rules
Branch protections will not automatically be copied from this repository. Ensure that you enable the following for the staging and production branches.
- **Require a pull request before merging**
- **Require approvals**
- **Do not allow bypassing the above settings**

## Understanding the CICD (Continuous Integration / Continuous Deployment) pipeline
This template includes a cicd.yml file in the .github/workflows folder that servers as the means for deploying infrastructure required for NASA-IMPACT projects. It is designed to take advantage of AWS's CDK and CloudFormation. As part of this workflow several steps take place. See below for an explanation of each.

### Gitflow enforcer:
This enforces Gitflow principles which is a branching model that involves the use of feature branches and multiple primary branches. For more information on Gitflow see [this](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow).

### Set environment:
This sets the environment based on the current branch and is used throughout the action. [Make sure that you have your environment variables and secrets populated properly](#populating-the-github-environments).

### CFN validation:
This step checks your CloudFormation template to make sure that the resources created by your CDK stack are in compliance with IMPACT rules. This is accomplished using CloudFormation Guard and a centrally managed rules file. If you would like to independently test your CloudFormation template please view [this](#checking-cloudformation-templates-independently). This step is designed to catch any incompliant resources before they are deployed in the UAH AWS environment. Skipping this step will return with an service control policy error during the following deploy step.

### Deploy
This step takes the validated CDK stack/CloudFormation template, assumes the appropriate role based on the current branch/environment, and deploys the stack to AWS.

## Populating the github environments
All secrets and variables should be handled using github environment secrets and variables accessed via the repository Settings and clicking **Environments** under "Code and automation". Here there should be a "production", "staging", and "development" environment to correspond with the three primary branches.

Below are some examples of what items should be stored in secrets vs variables.

Examples of items to store as github environment secrets:

  - AWS Account IDs
  - AWS Role ARNs
  - VPC IDs
  - Cognito Secrets
  - Security Group IDs
  - Any Sensitive Data (If you aren't sure, store it as a secret)

Examples of items to store as github environment variables:

  - File Paths
  - Configuration Parameters
  - Version Information
  - URLs and Endpoints
  - Feature Flags
  - Any Non-Sensitive Data

## Checking CloudFormation templates independently
It is recommended that while you develop your CDK stack/CloudFormation template that you run it against the UAH IMPACT environment rules to ensure that it is still in compliance. This can be done locally by doing the following:

1. Download and install CloudFormation Guard following [these instructions](https://docs.aws.amazon.com/cfn-guard/latest/ug/setting-up.html).
2. Download the central rules file [here](https://github.com/NASA-IMPACT/Lessons_Learned/blob/main/cfn-guard/enforce-tags.guard).
3. Synthesize your CDK stack using `aws cdk synth`
4. Run the following command.
```
cfn-guard validate \
--data <path/to/your/template> \
--rules <path/to/UAH/rules/file> \
--show-summary pass,fail \
--type CFNtemplate
```
