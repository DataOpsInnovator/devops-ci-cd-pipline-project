# DevOps CI/CD Infrastructure Pipeline

This repository contains Terraform infrastructure code and a Jenkins pipeline for provisioning AWS resources for a Python/Flask application.

## Pipeline behavior

- Checks out the repository configured for the Jenkins Pipeline job using checkout scm.
- Initializes Terraform in infra/.
- Runs a read-only plan when PLAN_TERRAFORM is selected.
- For APPLY_TERRAFORM, creates a saved plan and pauses for human approval before applying that exact plan.
- For DESTROY_TERRAFORM, creates a destroy plan and pauses for separate explicit approval before applying it.
- Disables concurrent pipeline runs to reduce the risk of Terraform state races.
- Removes saved plan files at the end of the run.

All operation parameters default to false. Do not enable apply or destroy without reviewing the plan, confirming the AWS account and region, and checking the Terraform state/backend configuration.

## Jenkins setup

1. Configure the Jenkins Pipeline job to use this repository from SCM.
2. Configure an AWS credential binding in Jenkins with the ID referenced by the Jenkinsfile (aws-crendentails-mgeta), or update that ID to match your team's credential naming convention.
3. Ensure the Jenkins agent has a compatible Terraform CLI and the permissions required for the intended infrastructure operations.
4. Review the Terraform configuration and backend/state settings before running the pipeline.

## Safety and scope

This pipeline can create or destroy billable cloud resources. The approval prompts are guardrails, not a substitute for reviewing the plan. Test in a disposable environment first. No claim is made here that the pipeline has been executed successfully against a live AWS account.
