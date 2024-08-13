pipeline {
    agent {
        kubernetes {
            label 'k8s'
            defaultContainer 'jnlp'
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins: slave
spec:
  containers:
  - name: docker
    image: docker:latest
    command:
    - cat
    tty: true
    volumeMounts:
    - name: docker-config
      mountPath: /root/.docker
  - name: kubectl
    image: bitnami/kubectl:latest
    command:
    - cat
    tty: true
  volumes:
  - name: docker-config
    configMap:
      name: docker-config
"""
        }
    }
    environment {
        PROJECT_ID = 'inftfy'
        CLUSTER_NAME = 'jenkins'
        LOCATION = 'asia-south1-c'
        CREDENTIALS_ID = 'git-jenkins'
    }
    stages {
        stage("Checkout code") {
            steps {
                checkout scm
            }
        }
        stage("Build image") {
            steps {
                container('docker') {
                    script {
                        myapp = docker.build("ashwini73/gke-jenkins:${env.BUILD_ID}")
                    }
                }
            }
        }
        stage("Push image") {
            steps {
                container('docker') {
                    script {
                        docker.withRegistry('https://index.docker.io/v1/', 'docker') {
                            myapp.push("latest")
                            myapp.push("${env.BUILD_ID}")
                        }
                    }
                }
            }
        }        
        stage('Deploy to GKE') {
            steps {
                container('kubectl') {
                    script {
                        // Update the deployment YAML with the new image tag
                        sh "sed -i 's/hello:latest/hello:${env.BUILD_ID}/g' deployment.yaml"

                        // Use kubectl to apply the updated YAML to the cluster
                        withKubeConfig(credentialsId: env.CREDENTIALS_ID) {
                            sh "kubectl apply -f deployment.yaml"
                        }
                    }
                }
            }
        }
    }
}
