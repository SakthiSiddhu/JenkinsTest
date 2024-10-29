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
                git branch: 'branch-name', url: 'https://github.com/SakthiSiddhu/SampleReactApp'
            }
        }

        stage('Build') {
            steps {
                // Command to build the project, skipping tests
                // Example: sh 'mvn clean install -DskipTests'
            }
        }

        stage('Docker Build and Push') {
            steps {
                script {
                    def dockerTag = 'latest' // or any other tag you want to use
                    def projectName = 'samplereactapp' // extracted from the GitHub URL

                    docker.build("your-dockerhub-username/${projectName}:${dockerTag}").push()

                    withCredentials([usernamePassword(credentialsId: 'your-credentials-id', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                        sh "docker push your-dockerhub-username/${projectName}:${dockerTag}"
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
                      name: ${projectName}-deployment
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
                            image: your-dockerhub-username/${projectName}:${dockerTag}
                            ports:
                            - containerPort: container-port
                    """

                    def serviceYaml = """
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: ${projectName}-service
                    spec:
                      selector:
                        app: ${projectName}
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
                sh 'kubectl port-forward --address 0.0.0.0 service/service-name container-port:service-port'
            }
        }
    }
}