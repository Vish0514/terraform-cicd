pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }
    stages {
        stage('checkout') {
            steps {
                script {
                    dir("terraform") {
                        git "https://github.com/Vish0514/terraform-cicd.git"
                    }
                }
            }
        }
        stage('validate') {
            steps {
                script {
                    sh 'terraform --version'
                    sh 'terraform init -backend-config="tfstate.config"'
                    sh 'terraform validate'
                }
            }
        }
        stage('plan') {
            steps {
                script {
                    dir("terraform") {
                        sh 'terraform init'
                        sh 'terraform plan -out=tfplan'
                        sh 'terraform show -no-color tfplan > tfplan.txt'
                    }
                }
            }
            post {
                success {
                    archiveArtifacts artifacts: 'terraform/tfplan', allowEmptyArchive: true
                }
            }
        }
        stage('apply') {
            when {
                expression {
                    currentBuild.result == null || currentBuild.result == 'SUCCESS'
                }
            }
            steps {
                input 'Proceed with Terraform apply?'
                script {
                    dir("terraform") {
                        sh 'terraform apply -input=false tfplan'
                    }
                }
            }
        }
        stage('destroy') {
            when {
                expression {
                    currentBuild.result == null || currentBuild.result == 'SUCCESS'
                }
            }
            steps {
                input 'Proceed with Terraform destroy?'
                script {
                    dir("terraform") {
                        sh 'terraform destroy --auto-approve'
                    }
                }
            }
        }
    }
}
