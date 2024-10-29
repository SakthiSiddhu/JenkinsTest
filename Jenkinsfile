pipeline {
    agent any

    tools {
        // Specify the tool name and type here
        // Example: maven 'Maven 3.6.3'
    }

    environment {
        KUBECONFIG = 'path/to/kubeconfig'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'branch-name', url: 'https://github.com/SakthiSiddhu/TodoBackend'
            }
        }

        stage('Build') {
            steps {
                // Use the appropriate build command for your project
                // Example: sh 'mvn clean install -DskipTests'
            }
        }

        stage('Docker Build and Push') {
            steps {
                script {
                    def projectName = 'todobackend'
                    def dockerTag = 'dockerTag'
                    def dockerImage = "dockerhub-username/${projectName}:${dockerTag}"

                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials-id', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh """
                            docker build -t ${dockerImage} .
                            echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin
                            docker push ${dockerImage}
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def projectName = 'todobackend'
                    def dockerTag = 'dockerTag'
                    def dockerImage = "dockerhub-username/${projectName}:${dockerTag}"
                    def containerPort = 'container-port'
                    def servicePort = 'service-port'

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
                            image: ${dockerImage}
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
                    def serviceName = 'todobackend'
                    def containerPort = 'container-port'
                    def servicePort = 'service-port'

                    sh "kubectl port-forward --address 0.0.0.0 service/${serviceName} ${containerPort}:${servicePort}"
                }
            }
        }
    }
}