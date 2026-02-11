pipeline {
    agent any

    stages {
        stage('Deploy To Kubernetes') {
            steps {
                   withKubeConfig(caCertificate: '', clusterName: 'mycluster', contextName: '', credentialsId: 'k8s-cred', namespace: 'microservice', restrictKubeConfigAccess: false, serverUrl: 'https://8A8B0306DC014B768F992F048FC9C439.gr7.us-east-1.eks.amazonaws.com') {
                    sh "kubectl apply -f deployment-service.yml "
                    
                }
            }
        }
        
        stage('verify Deployment') {
            steps {
                  withKubeConfig(caCertificate: '', clusterName: 'mycluster', contextName: '', credentialsId: 'k8s-cred', namespace: 'microservice', restrictKubeConfigAccess: false, serverUrl: 'https://8A8B0306DC014B768F992F048FC9C439.gr7.us-east-1.eks.amazonaws.com') {  
                  sh "kubectl get svc -n microservice"
                }
            }
        }
    }
}
