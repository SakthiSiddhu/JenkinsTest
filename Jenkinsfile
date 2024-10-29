pipeline {
    agent any

    tools {
        maven 'Maven_3.9.9'
    }

    environment {
        KUBECONFIG = '/home/ec2-user/.kube/config'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/SakthiSiddhu/TodoBackend'
            }
        }

        stage('Build Maven Project') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def dockerTag = "latest"
                    def projectName = "todohub"

                    sh "docker build -t sakthisiddu1/${projectName}:$dockerTag ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhubpwd', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    script {
                        def dockerTag = "latest"
                        def projectName = "todohub"

                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                        sh "docker push sakthisiddu1/${projectName}:$dockerTag"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def dockerTag = "latest"
                    def projectName = "todohub"
                    def deploymentYAML = """
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
                            image: sakthisiddu1/${projectName}:${dockerTag}
                            ports:
                            - containerPort: 9000
                    """

                    def serviceYAML = """
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: ${projectName}-service
                    spec:
                      type: NodePort
                      selector:
                        app: ${projectName}
                      ports:
                        - protocol: TCP
                          port: 9000
                          targetPort: 9000
                    """

                    writeFile file: 'deployment.yaml', text: deploymentYAML
                    writeFile file: 'service.yaml', text: serviceYAML

                    sh "kubectl apply -f deployment.yaml"
                    sh "kubectl apply -f service.yaml"
                    }
                }
            }
        }
    }
}