pipeline {
    agent any

    stages {
        stage('Configure') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Madhu-gt-hb/MyRepo.git']])
            }
        }
        stage('Building image') {
            steps {
                sh 'docker build -t word-counter .'
            }
        }
        
        stage('Pushing to ECR') {
            steps {
                withAWS(credentials: 'AWS-CREDS', region: 'ap-south-2') {
                    sh 'aws ecr get-login-password --region ap-south-2 | docker login --username AWS --password-stdin 578257747988.dkr.ecr.ap-south-2.amazonaws.com'
                    sh 'docker tag word-counter:latest 578257747988.dkr.ecr.ap-south-2.amazonaws.com/word-counter:latest'
                    sh 'docker push 578257747988.dkr.ecr.ap-south-2.amazonaws.com/word-counter:latest'
                }
            }
        }
          
        stage('K8S Deploy') {
            steps {
                script {
                    withAWS(credentials: 'AWS-CREDS', region: 'ap-south-2') {
                        sh 'aws eks update-kubeconfig --name k8-1 --region ap-south-2'
                        sh 'kubectl apply -f EKS-Deployment.yaml'
                    }
                }
            }
        }

        stage('Get Service URL') {
            steps {
                script {
                    def serviceUrl = ""
                    // Wait for the LoadBalancer IP to be assigned
                    timeout(time: 5, unit: 'MINUTES') {
                        while(serviceUrl == "") {
                            serviceUrl = sh(script: "kubectl get svc word-counter-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'", returnStdout: true).trim()
                            if(serviceUrl == "") {
                                echo "Waiting for the LoadBalancer IP..."
                                sleep 10
                            }
                        }
                    }
                    echo "Service URL: http://${serviceUrl}"
                }
            }
        }
    }
}
