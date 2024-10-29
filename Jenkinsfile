```groovy
pipeline {
    agent any

    tools {
        maven 'Maven_3.9.9'
    }
    
    environment {
        KUBECONFIG = "/home/ec2-user/.kube/config"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/SakthiSiddhu/TodoBackend'
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
                    def projectName = 'todobackend'
                    def dockerTag = "latest"
                    def dockerImage = "sakthisiddu1/${projectName}:${dockerTag}"
                    
                    sh "docker build -t ${dockerImage} ."
                }
            }
        }
        
        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhubpwd', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                        sh "echo $DOCKER_HUB_PASSWORD | docker login -u $DOCKER_HUB_USERNAME --password-stdin"
                        def projectName = 'todobackend'
                        def dockerTag = "latest"
                        def dockerImage = "sakthisiddu1/${projectName}:${dockerTag}"
                        
                        sh "docker push ${dockerImage}"
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deploymentYAML = '''
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
        image: sakthisiddu1/todobackend:latest
        ports:
        - containerPort: 9000
                    '''
                    
                    def serviceYAML = '''
apiVersion: v1
kind: Service
metadata:
  name: todobackend-service
spec:
  selector:
    app: todobackend
  ports:
    - protocol: TCP
      port: 9000
      targetPort: 9000
                    '''
                    
                    writeFile file: 'deployment.yaml', text: deploymentYAML
                    writeFile file: 'service.yaml', text: serviceYAML
                    
                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
                }
            }
        }
    }
}
```