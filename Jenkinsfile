pipeline {
    agent any

    options {
        // Prevent overlapping runs from racing on the same Terraform state.
        disableConcurrentBuilds()
    }

    parameters {
        booleanParam(name: 'PLAN_TERRAFORM', defaultValue: false, description: 'Run a read-only Terraform plan')
        booleanParam(name: 'APPLY_TERRAFORM', defaultValue: false, description: 'Plan, request approval, then apply Terraform changes')
        booleanParam(name: 'DESTROY_TERRAFORM', defaultValue: false, description: 'Plan destruction, request explicit approval, then destroy resources')
    }

    stages {
        stage('Validate Parameters') {
            steps {
                script {
                    if (params.APPLY_TERRAFORM && params.DESTROY_TERRAFORM) {
                        error('Choose either APPLY_TERRAFORM or DESTROY_TERRAFORM, not both in the same run.')
                    }
                }
            }
        }

        stage('Checkout Repository') {
            steps {
                // Jenkins Pipeline-from-SCM already knows the configured repository.
                // Keep that source of truth rather than deleting the workspace and
                // cloning a hard-coded repository URL.
                checkout scm
                sh 'pwd && ls -la'
            }
        }

        stage('Terraform Init') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-crendentails-mgeta']]) {
                    dir('infra') {
                        sh 'terraform init'
                    }
                }
            }
        }

        stage('Terraform Plan') {
            when {
                expression { params.PLAN_TERRAFORM }
            }
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-crendentails-mgeta']]) {
                    dir('infra') {
                        sh 'terraform plan -input=false'
                    }
                }
            }
        }

        stage('Terraform Apply') {
            when {
                expression { params.APPLY_TERRAFORM }
            }
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-crendentails-mgeta']]) {
                    dir('infra') {
                        sh 'terraform plan -input=false -out=tfplan'
                    }
                }
                // Do not hold cloud credentials in the environment while waiting for approval.
                input message: 'Review the Terraform plan in the build log. Apply these infrastructure changes?', ok: 'Approve apply'
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-crendentails-mgeta']]) {
                    dir('infra') {
                        sh 'terraform apply -input=false tfplan'
                    }
                }
            }
        }

        stage('Terraform Destroy') {
            when {
                expression { params.DESTROY_TERRAFORM }
            }
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-crendentails-mgeta']]) {
                    dir('infra') {
                        sh 'terraform plan -destroy -input=false -out=tfplan-destroy'
                    }
                }
                // Do not hold cloud credentials in the environment while waiting for approval.
                input message: 'This will destroy Terraform-managed infrastructure. Confirm the plan and resource scope before proceeding.', ok: 'Approve destroy'
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-crendentails-mgeta']]) {
                    dir('infra') {
                        sh 'terraform apply -input=false tfplan-destroy'
                    }
                }
            }
        }
    }

    post {
        always {
            // Saved plans can contain sensitive values; do not retain them in the workspace.
            if (fileExists('infra')) {
                dir('infra') {
                    sh 'rm -f tfplan tfplan-destroy'
                }
            }
        }
    }
}