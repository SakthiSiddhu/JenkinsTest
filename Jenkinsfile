pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        KUBECONFIG = '/path/to/kubeconfig'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/SakthiSiddhu/TodoBackend'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def dockerTag = "latest"
                    def projectName = "todobackend"
                    sh "docker build -t ${projectName}:${dockerTag} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    script {
                        def dockerTag = "latest"
                        def projectName = "todobackend"
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                        sh "docker tag ${projectName}:${dockerTag} $DOCKER_USERNAME/${projectName}:${dockerTag}"
                        sh "docker push $DOCKER_USERNAME/${projectName}:${dockerTag}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def projectName = "todobackend"
                    def dockerTag = "latest"
                    def containerPort = 8080
                    def servicePort = 80

                    writeFile file: 'deployment.yaml', text: """
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: ${projectName}
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: ${projectName}
                      template:
                        metadata:
                          labels:
                            app: ${projectName}
                        spec:
                          containers:
                          - name: ${projectName}
                            image: $DOCKER_USERNAME/${projectName}:${dockerTag}
                            ports:
                            - containerPort: ${containerPort}
                    """

                    writeFile file: 'service.yaml', text: """
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: ${projectName}
                    spec:
                      selector:
                        app: ${projectName}
                      ports:
                        - protocol: TCP
                          port: ${servicePort}
                          targetPort: ${containerPort}
                    """

                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
                }
            }
        }

        stage('Sleep') {
            steps {
                sleep time: 1, unit: 'MINUTES'
            }
        }

        stage('Port Forward') {
            steps {
                script {
                    def projectName = "todobackend"
                    def containerPort = 8080
                    def servicePort = 80
                    sh "kubectl port-forward --address 0.0.0.0 service/${projectName} ${containerPort}:${servicePort}"
                }
            }
        }
    }
}