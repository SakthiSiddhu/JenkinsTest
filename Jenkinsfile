pipeline {
    agent any

    tools {
        // Specify the tool name and type here
        // Example: maven 'M3'
    }

    environment {
        KUBECONFIG = 'path/to/kubeconfig'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/SakthiSiddhu/TodoBackend'
            }
        }

        stage('Build') {
            steps {
                // Replace with the appropriate build command
                // Example: sh 'mvn clean install -DskipTests'
            }
        }

        stage('Docker Build and Push') {
            steps {
                script {
                    def projectName = 'todobackend'
                    def dockerTag = 'latest'
                    def dockerImage = "${projectName}:${dockerTag}"

                    sh "docker build -t ${dockerImage} ."

                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                        sh "docker tag ${dockerImage} ${DOCKER_USERNAME}/${dockerImage}"
                        sh "docker push ${DOCKER_USERNAME}/${dockerImage}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deploymentYaml = """
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: todobackend-deployment
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: todobackend
                      template:
                        metadata:
                          labels:
                            app: todobackend
                        spec:
                          containers:
                          - name: todobackend
                            image: ${DOCKER_USERNAME}/todobackend:latest
                            ports:
                            - containerPort: 8080
                    """

                    def serviceYaml = """
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: todobackend-service
                    spec:
                      selector:
                        app: todobackend
                      ports:
                      - protocol: TCP
                        port: 80
                        targetPort: 8080
                    """

                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml

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
                sh 'kubectl port-forward --address 0.0.0.0 service/todobackend-service 8080:80'
            }
        }
    }
}