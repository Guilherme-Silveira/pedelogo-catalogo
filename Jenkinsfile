pipeline {
    agent any

    stages {
        stage('Git Checkout') {
            steps {
                git url: 'https://github.com/Guilherme-Silveira/pedelogo-catalogo.git', branch: 'main'
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    dockerapp = docker.build("guisilveira/pedelogo-catalogo:${env.BUILD_ID}", "-f ./src/PedeLogo.Catalogo.Api/Dockerfile .")
                }
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        dockerapp.push('latest')
                        dockerapp.push("${env.BUILD_ID}")
                    }  
                } 
            }
        }
        stage('Deploy Kubernetes') {
            agent {
                kubernetes {
                    cloud 'kubernetes'
                }
            }
            steps {
                sh 'apt update'
                sh 'apt install curl'
                sh 'curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" -o /tmp/kubectl'
                sh 'chmod +x /tmp/kubectl'
                withKubeConfig([credentialsId: 'kubeconfig', serverUrl: 'https://k3d-silveira-server-0:6443']) {
                    sh '/tmp/kubectl apply -f ./k8s'
                }
            }
        }
    }
}