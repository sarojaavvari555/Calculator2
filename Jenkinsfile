pipeline {
    agent any

   stages {
        stage('code checkout') {
            steps {
                // Get some code from a GitHub repository
                git 'https://github.com/fayazdev/Calculator.git'
            }
        }
        stage('Maven Build') {
            steps {
                
                // Run Maven on a Unix agent.
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(installationName: 'SonarQube', credentialsId: 'sonar-token') {
                    sh "mvn clean verify sonar:sonar -Dsonar.projectKey=Calculator -Dsonar.projectName='Calculator'"
                }    
            } 
        }
        stage('Build docker') {
            steps {
                //sh "sudo docker build -t calculator . --file dockerfile"
                sh '''#!/bin/bash
                docker build -t calculator . --file Dockerfile
                '''
            }
        }
        stage('AWS Authentication') {
            steps {
                withAWS(region: 'ap-south-1', credentials: 'aws-credentials') {
                    sh "aws eks --region us-east-1 update-kubeconfig --name demo-eks"
                }
            }
        }
        stage('Kubernetes Setup') {
            steps {
                // Use Kubernetes plugin to set up the connection to EKS
                withKubeConfig(credentialsId: 'K8S', clusterName: 'eks', serverUrl: 'https://B5FE4869B0809F47FCBB3CAE5F9E14D8.gr7.us-east-1.eks.amazonaws.com') {
                }
            }
        }
        stage('Deploy app in kubernetes') {
            steps {
                sh 'kubectl delete ns nginx'
                sleep 30
                sh 'kubectl create namespace nginx'
                sh 'kubectl get ns'
                
                // Add Helm repository if necessary
                sh 'kubectl apply -f deployment.yaml -n nginx'
                
                // Install Helm chart
                sh 'kubectl apply -f service.yaml -n nginx'
                
                sh 'kubectl apply -f nginx-loadbalancer.yaml -n nginx'
                
            }
        }
        stage('verify') {
            steps {
                // display the cluster nodes
                sh 'kubectl get nodes'
                
                // disply the deployed pods
                sh 'kubectl get pods -n nginx'
                
                sh 'kubectl get svc -n nginx'
            }
        }
    }
}
