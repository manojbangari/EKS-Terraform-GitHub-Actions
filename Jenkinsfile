properties([
    parameters([
        string(defaultValue: 'dev', name: 'Environment'),
        choice(choices: ['plan', 'apply', 'destroy'], name: 'Terraform_Action')
    ])
])

pipeline {
    agent {
        kubernetes {
            label 'eks-agent'
            defaultContainer 'jnlp'    
            yamlFile 'kubernetespod.yaml'
        }
    }

    stages {
        stage('Git Checkout') {
            steps {
                container('jnlp') {                       
                    git branch: 'master', 
                        url: 'https://github.com/manojbangari/EKS-Terraform-GitHub-Actions.git'
                }
            }
        }

        stage('Terraform Init') {
            steps {
                container('terraform') {                
                    withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                        sh 'terraform -chdir=eks init'
                    }
                }
            }
        }

        stage('Terraform Validate') {
            steps {
                container('terraform') {
                    withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                        sh 'terraform -chdir=eks validate'
                    }
                }
            }
        }

        stage('Terraform Action') {
            steps {
                container('terraform') {
                    withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                        script {
                            if (params.Terraform_Action == 'plan') {
                                sh "terraform -chdir=eks plan -var-file=${params.Environment}.tfvars"
                            } else if (params.Terraform_Action == 'apply') {
                                sh "terraform -chdir=eks apply -var-file=${params.Environment}.tfvars -auto-approve"
                            } else if (params.Terraform_Action == 'destroy') {
                                sh "terraform -chdir=eks destroy -var-file=${params.Environment}.tfvars -auto-approve"
                            }
                        }
                    }
                }
            }
        }
    }
}
