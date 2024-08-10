pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "your-dockerhub-username/helloworldapp"
        DOCKER_REGISTRY = "docker.io" // Docker Hub registry
    }

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'github-credentials', url: 'https://github.com/your-username/your-repo.git'
            }
        }

        stage('Build') {
            steps {
                script {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Dockerize') {
            steps {
                script {
                    sh 'docker build -t $DOCKER_IMAGE .'
                    sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    sh '''
                    kubectl apply -f - <<EOF
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: helloworldapp-deployment
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: helloworldapp
                      template:
                        metadata:
                          labels:
                            app: helloworldapp
                        spec:
                          containers:
                          - name: helloworldapp
                            image: $DOCKER_IMAGE
                            ports:
                            - containerPort: 8080
                    EOF
                    '''

                    sh '''
                    kubectl apply -f - <<EOF
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: helloworldapp-service
                    spec:
                      selector:
                        app: helloworldapp
                      ports:
                      - protocol: TCP
                        port: 8181
                        targetPort: 8080
                      type: NodePort
                    EOF
                    '''
                }
            }
        }
    }

    environment {
        DOCKER_USERNAME = credentials('dockerhub-username') // Jenkins credentials ID for Docker Hub username
        DOCKER_PASSWORD = credentials('dockerhub-password') // Jenkins credentials ID for Docker Hub password
    }

    post {
        always {
            cleanWs()
        }
    }
}