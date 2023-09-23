pipeline {
    agent any
    
    stages {
        stage('Fetch code') {
            steps {
                git 'https://github.com/PranitRout07/django_crud.git'
            }
        }
        stage('Code Analysis') {
            environment {
                scannerHome = tool 'sonar4.7'
            }
            steps {
               withSonarQubeEnv('sonar') {
                   bat '%scannerHome%\\bin\\sonar-scanner -Dsonar.projectKey=crud-app ' +
                    '-Dsonar.projectName=crud-app ' +
                    '-Dsonar.projectVersion=1.0 ' +
                    '-Dsonar.sources=. '
              }
            }
        }
        stage('Docker build image'){
            steps{
                dir(path: 'apps') {
                    bat 'docker build -t crud-app .'
                }
            }
        }
        stage('Push docker image to dockerhub'){
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerPass', usernameVariable: 'dockerUser')]) {
                   bat "docker login -u ${env.dockerUser} -p ${env.dockerPass}"
                   bat "docker tag crud-app ${env.dockerUser}/crud-app:latest"
                   bat "docker push ${env.dockerUser}/crud-app:latest "
                }
            }
        }
        stage('scan with trivy') {
            steps {
            
                bat "docker run --rm -v D:/trivy-report/:/root/.cache/ aquasec/trivy:0.18.3 image pranit007/crud-app > reports"
            }
        }
        stage('Run minikube'){
            steps{
                
                bat 'minikube start'
            }
        }

        stage('run deploy and service'){
            steps{

                bat 'kubectl apply -f deployment.yml'
                bat 'kubectl apply -f service.yml'
                
                
            }
            
        }
        stage('Introduce a 30-Second Delay') {
            steps {
                // Sleep for 30 seconds
                sleep time: 30, unit: 'SECONDS'
            }
        }
        stage('Run the service') {
            steps {
                bat 'minikube service my-service --url'
            }
        }
        
    }
}
