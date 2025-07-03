pipeline {
    agent any

    parameters {
        choice(
            name: 'TERRAFORM_ACTION',
            choices: ['apply', 'destroy'],
            description: 'Select Terraform action: apply to provision, destroy to tear down'
        )
    }

    environment {
        // Add your AWS credentials or use instance profile
        // AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
        // AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        // Optionally set AWS_DEFAULT_REGION
    }

    options {
        disableConcurrentBuilds()
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
                cleanWs()
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                sh 'terraform init -input=false'
            }
        }

        stage('Terraform Validate') {
            when {
                expression { params.TERRAFORM_ACTION == 'apply' }
            }
            steps {
                sh 'terraform validate'
            }
        }

        stage('Terraform Plan') {
            when {
                expression { params.TERRAFORM_ACTION == 'apply' }
            }
            steps {
                sh 'terraform plan -out=tfplan'
                archiveArtifacts artifacts: 'tfplan', onlyIfSuccessful: true
            }
        }

        stage('Manual Approval') {
            when {
                expression { params.TERRAFORM_ACTION == 'destroy' }
            }
            steps {
                script {
                    input message: 'Are you absolutely sure you want to destroy all resources?'
                }
            }
        }

        stage('Terraform Action') {
            steps {
                script {
                    if (params.TERRAFORM_ACTION == 'apply') {
                        sh 'terraform apply -auto-approve tfplan'
                    } else if (params.TERRAFORM_ACTION == 'destroy') {
                        // Backup state file before destroy (best practice)
                        sh 'cp terraform.tfstate terraform.tfstate.backup || true'
                        sh 'terraform destroy -auto-approve'
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '*.tfstate*', onlyIfSuccessful: false
            cleanWs()
        }
        failure {
            mail to: 'devops-team@example.com',
                 subject: "Jenkins Terraform Pipeline FAILED: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                 body: "Check the Jenkins job for details."
        }
    }
}
