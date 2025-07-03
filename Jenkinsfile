pipeline {
    agent any

    environment {
        // Set your AWS region
        AWS_DEFAULT_REGION = 'us-east-1'
        // Add more environment variables if needed
    }

    options {
        // Keep only the last 10 builds to save space
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('Checkout') {
            steps {
                // Jenkins will automatically check out the repo if using Pipeline from SCM,
                // but this is explicit for clarity.
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }

        stage('Terraform Validate') {
            steps {
                sh 'terraform validate'
            }
        }

        stage('Terraform Plan') {
            steps {
                sh 'terraform plan -out=tfplan'
                // Optionally, archive the plan file
                archiveArtifacts artifacts: 'tfplan', allowEmptyArchive: true
            }
        }

        stage('Terraform Apply') {
            steps {
                // Optional: Add manual approval before apply in production
                input(message: 'Approve to apply Terraform changes?')
                sh 'terraform apply -auto-approve tfplan'
            }
        }
    }

    post {
        always {
            // Archive Terraform state files for reference (do NOT use this for state backup)
            archiveArtifacts artifacts: '**/*.tfstate', allowEmptyArchive: true
            // Clean up plan file
            sh 'rm -f tfplan'
        }
        failure {
            mail to: 'your-team@example.com',
                 subject: "Jenkins Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Check Jenkins for details: ${env.BUILD_URL}"
        }
    }
}
