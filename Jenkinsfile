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
                    yaml '''
                        apiVersion: v1
                        kind: Pod
                        spec:
                            containers:
                            - name: slave-k8s
                              image: guisilveira/slave-k8s
                              command: 
                                - cat
                              tty: true
                    '''
                }
            }
            steps {
                    withKubeConfig([credentialsId: 'kubeconfig', serverUrl: 'https://k3d-silveira-server-0:6443']) {
                        sh 'kubectl apply -f ./k8s'
                    
                }
            }
        }
    }
}