def genaralvars () {

    env.GIT_REPO = 'https://gitlab.com/chaloglez/labsacademia.git'
    env.GIT_BRANCH = 'ansiblelab01'
    env.DOCKER_REPO = 'gonzafirma'
    CONTAINER_PORT= '80'

}

pipeline {
    agent any
    tools {
       terraform 'terraform-2'
    }
    stages {
        stage ("Set Variables") {
            steps {                
                genaralvars()
            }
        }
        stage ("Get Code") {
            steps {
                git branch: "${env.GIT_BRANCH}", url: "${env.GIT_REPO}"
            }
        }
        stage ("Verify If exist container") {
            steps {
                    script {
                        if (fileExists('terraform.tfstate')) {
                            withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-gonzafirma', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]){
                                sh "terraform destroy --auto-approve"
                            }
                        }
                        else {
                            sh "echo no existe tfstate"
                        }
                }
            }
        }
        stage('terraform format check') {
            steps{
                sh 'terraform fmt'
            }
        }
        stage('terraform Init') {
            steps{
                sh 'terraform init'
            }
        }
        stage('terraform apply') {
            steps{
                //withAWS(credentials: 'aws-gonzafirma', region: 'us-east-1') {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-gonzafirma', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]){
                    sh "terraform apply --auto-approve"
                }
                script {
                    PUBLIC_IP_EC2 = sh (script: "terraform output instance_public_ip", returnStdout:true).trim()
                }
                echo "${PUBLIC_IP_EC2}"
            }
        }
        stage('Change inventory content') {
            steps{
                sh "echo $PUBLIC_IP_EC2 > inventory.hosts"
            }
        }     
        stage('Wait 5 minutes') {
            steps {
                sleep time:5, unit: 'MINUTES'
            }
        }
        stage ("Ansible Hello World") {
            steps {
                ansiblePlaybook become: true, colorized: true, extras: '-v', disableHostKeyChecking: true, credentialsId: 'gonzafirma-ssh-server01', installation: 'ansible210', inventory: 'inventory.hosts', playbook: 'playbook-hello-world.yml'
            }
        }
        
        stage('Manual Approval to Destroy the Infra') {
            steps{
                input "Proceed with destroy the Infra?"
            }
        }
        stage('Executing Terraform Destroy') {
            steps{
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-gonzafirma', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]){
                sh "terraform destroy --auto-approve"
            }
            }
        }
    }
}
