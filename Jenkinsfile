def genaralvars () {
    env.GIT_REPO = 'https://gitlab.com/chaloglez/labsacademia.git'
    env.GIT_BRANCH = 'main'
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
        //stage ("Get Code") {
        //    steps {
        //        git branch: "${env.GIT_BRANCH}", url: "${env.GIT_REPO}"
        //    }
        //}
        stage ("Verify If exist container") {
            steps {
                    script {
                        DOCKERID = sh (script: "docker ps -f publish=${CONTAINER_PORT} -q", returnStdout: true).trim()
                        if  ( DOCKERID !="" ) {
                            if (fileExists('terraform.tfstate')) {
                                sh "terraform destroy  --target docker_container.nginx -var=\"container_port=${CONTAINER_PORT}\" -var=\"reponame=${env.DOCKER_REPO}\" --auto-approve"
                            }
                            else {
                                sh "docker stop ${DOCKERID}"
                            }
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
                sh "terraform apply -var=\"container_port=${CONTAINER_PORT}\" -var=\"reponame=${env.DOCKER_REPO}\" --auto-approve"
            }
        }
        stage('Manual Approval to Destroy the Infra') {
            steps{
                input "Proceed with destroy the Infra?"
            }
        }
        stage('Executing Terraform Destroy') {
            steps{
                sh "terraform destroy --target docker_container.nginx -var=\"container_port=${CONTAINER_PORT}\" -var=\"reponame=${env.DOCKER_REPO}\" --auto-approve"
            }
        }
    }

    
}
