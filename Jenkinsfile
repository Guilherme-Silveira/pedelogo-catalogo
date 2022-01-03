pipeline {
    agent {
        kubernetes {
            yaml '''
                apiVersion: v1
                kind: Pod
                spec:
                    containers:
                    - name: docker
                      image: docker:dind
                      securityContext:
                        privileged: true
                      env:
                      - name: DOCKER_TLS_CERTDIR
                        value: ""
                    - name: kubectl
                      image: guisilveira/kubectl
                      command:
                        - cat
                      tty: true 
            '''
        }
    }

    stages {
        stage('Git Checkout') {
            steps {
                container('docker') {
                    git url: 'https://github.com/Guilherme-Silveira/pedelogo-catalogo.git', branch: 'main'
                }
            }
        }
        stage('Docker Build') {
            steps {
                container('docker') {
                    script {
                        dockerapp = docker.build("guisilveira/pedelogo-catalogo:${env.BUILD_ID}", "-f ./src/PedeLogo.Catalogo.Api/Dockerfile .")
                    }
                }
            }
        }
        stage('Docker Push') {
            steps {
                container('docker') {
                    script {
                        docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                            dockerapp.push('latest')
                            dockerapp.push("${env.BUILD_ID}")
                        }  
                    } 
                }
            }
        }
        stage('Deploy Kubernetes') {
            steps {
                container('kubectl') {
                    git url: 'https://github.com/Guilherme-Silveira/pedelogo-catalogo.git', branch: 'main'
                    withKubeConfig([credentialsId: 'kubeconfig', serverUrl: 'https://k3d-silveira-server-0:6443']) {
                        sh 'kubectl apply -f ./k8s'
                    }
                }
            }
        }
    }
}