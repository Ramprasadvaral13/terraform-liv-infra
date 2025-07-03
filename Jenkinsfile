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
        AWS_DEFAULT_REGION = 'us-east-1'
        // Add more environment variables or credentials as needed
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
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
                archiveArtifacts artifacts: 'tfplan', allowEmptyArchive: true
            }
        }

        stage('Manual Approval for Destroy') {
            when {
                expression { params.TERRAFORM_ACTION == 'destroy' }
            }
            steps {
                input(message: 'Are you absolutely sure you want to destroy ALL resources?')
            }
        }

        stage('Terraform Action') {
            steps {
                script {
                    if (params.TERRAFORM_ACTION == 'apply') {
                        sh 'terraform apply -auto-approve tfplan'
                    } else if (params.TERRAFORM_ACTION == 'destroy') {
                        // Optional: backup state before destroy
                        sh 'cp terraform.tfstate terraform.tfstate.backup || true'
                        sh 'terraform destroy -auto-approve'
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/*.tfstate*', allowEmptyArchive: true
            sh 'rm -f tfplan || true'
            cleanWs()
        }
        failure {
            mail to: 'your-team@example.com',
                 subject: "Jenkins Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Check Jenkins for details: ${env.BUILD_URL}"
        }
    }
}
