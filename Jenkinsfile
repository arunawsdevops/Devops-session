pipeline {
    agent any

    parameters {
        choice(name: 'TERRAFORM_ACTION', choices: ['apply', 'destroy', 'plan'], description: 'Select Terraform action to perform')
        string(name: 'USER_NAME', defaultValue: 'Arun', description: 'Specify who is running the code')
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        AWS_DEFAULT_REGION    = 'us-east-1'
        TF_VAR_aws_region     = 'us-east-1'  // Set Terraform variable
    }

    stages {
        stage('Checkout') {
            steps {
                script {                 
                    withCredentials([string(credentialsId: 'github-pat', variable: 'GITHUB_PAT')]) {
                        sh "git clone https://$GITHUB_PAT@github.com/arunawsdevops/EKS-Cluster-terraform.git"                   
                    }
                }
            }
        }

        stage('Terraform Init') {
            steps {
                // Initialize Terraform
                // Use withCredentials for AWS credentials
                dir('EKS-Cluster-terraform') {
                    withCredentials([string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'), 
                                     string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh 'terraform init'
                    }
                }
            }
        }

        stage('Terraform Action') {
            steps {
                script {
                    // Perform selected Terraform action
                    dir('EKS-Cluster-terraform') {
                        if (params.TERRAFORM_ACTION == 'apply') {
                            sh 'terraform apply -auto-approve'
                        } else if (params.TERRAFORM_ACTION == 'destroy') {
                            sh 'terraform destroy -auto-approve'
                        } else if (params.TERRAFORM_ACTION == 'plan') {
                            sh 'terraform plan'
                        } else {
                            error "Invalid Terraform action selected: ${params.TERRAFORM_ACTION}"
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up workspace after pipeline execution
            cleanWs()
        }
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
}
