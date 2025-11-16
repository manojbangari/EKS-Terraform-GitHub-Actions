properties([
    parameters([
        string(
            defaultValue: 'dev',
            name: 'Environment'
        ),
        choice(
            choices: ['plan', 'apply', 'destroy'], 
            name: 'Terraform_Action'
        )])
])
pipeline {
    agent {
        kubernetes {
            label 'eks-agent'
            // REMOVED: yamlFile 'kubernetespod.yaml'
            // ADDED: The full Pod definition inline (using the corrected 'sleep' command)
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins/label: "eks-agent"
spec:
  serviceAccountName: jenkins
  
  containers:
  # The mandatory Jenkins Inbound Agent container
  - name: jnlp
    image: jenkins/inbound-agent:latest
    # Ensure JNLP has enough memory to run the Jenkins agent process
    resources:
      requests:
        cpu: '100m'
        memory: '256Mi'

  # The custom container for Node.js builds
  - name: node-builder
    image: node:18-alpine
    # Robust command to keep the container alive
    command: ["/bin/sh"] 
    args: ["-c", "trap 'exit 0' TERM INT; sleep infinity & wait"]
    resources:
      limits:
        cpu: '1'
        memory: '512Mi'
      requests:
        cpu: '250m'
        memory: '256Mi'
  
  volumes:
  - name: build-cache
    emptyDir: {}
"""
        }
    }
    stages {
        stage('Test EKS Pod') {
            steps {
                // Run the steps inside the 'node-builder' container
                container('node-builder') {
                    // Use the shared volume to cache node_modules
                    sh 'mkdir -p /home/jenkins/agent/my-app/node_modules_cache'
                    sh 'cp -r node_modules /home/jenkins/agent/my-app/node_modules_cache || true'
                    // Simulate an npm install for a simple app
                    sh 'echo "Installing dependencies..." && sleep 5'
                    }
            }
        }
        stage('Git Pulling') {

            steps {
                git branch: 'master', url: 'https://github.com/manojbangari/EKS-Terraform-GitHub-Actions.git'
            }
        }
        stage('Init') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                sh 'terraform -chdir=eks/ init'
                }
            }
        }
        stage('Validate') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                sh 'terraform -chdir=eks/ validate'
                }
            }
        }
        stage('Action') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    script {    
                        if (params.Terraform_Action == 'plan') {
                            sh "terraform -chdir=eks/ plan -var-file=${params.Environment}.tfvars"
                        }   else if (params.Terraform_Action == 'apply') {
                            sh "terraform -chdir=eks/ apply -var-file=${params.Environment}.tfvars -auto-approve"
                        }   else if (params.Terraform_Action == 'destroy') {
                            sh "terraform -chdir=eks/ destroy -var-file=${params.Environment}.tfvars -auto-approve"
                        } else {
                            error "Invalid value for Terraform_Action: ${params.Terraform_Action}"
                        }
                    }
                }
            }
        }
    }
}
