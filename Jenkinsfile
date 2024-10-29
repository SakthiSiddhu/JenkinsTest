pipeline {
    agent any
    tools {
        // Specify the tool name and type here
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
                // Command to build the project, skipping tests
            }
        }
        stage('Docker Build and Push') {
            steps {
                script {
                    def projectName = 'todotbackend'
                    def dockerTag = 'dockerTag'
                    def dockerImage = "${projectName}:${dockerTag}"
                    
                    sh "docker build -t ${dockerImage} ."
                    
                    withCredentials([usernamePassword(credentialsId: 'credentials-id', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                        sh "docker push ${dockerImage}"
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
                      name: todotbackend-deployment
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: todotbackend
                      template:
                        metadata:
                          labels:
                            app: todotbackend
                        spec:
                          containers:
                          - name: todotbackend
                            image: todotbackend:dockerTag
                            ports:
                            - containerPort: container-port
                    """
                    
                    def serviceYaml = """
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: todotbackend-service
                    spec:
                      selector:
                        app: todotbackend
                      ports:
                      - protocol: TCP
                        port: service-port
                        targetPort: container-port
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
                sh 'kubectl port-forward --address 0.0.0.0 service/todotbackend-service container-port:service-port'
            }
        }
    }
}