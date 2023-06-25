pipeline {
    agent any
    environment {
        INFRA_FLASK_VERSION = '1.0.0'
        FLASK_APP_VERSION = '1.0.0'
    }
    stages {
        stage('git clone') {
            steps {
                sh 'rm -rf *'
                sh 'git clone https://github.com/sivanmarom/final_project.git'
            }
        }

        stage('build and test infra_flask') {
            steps {
                dir('/var/lib/jenkins/workspace/deployment/final_project/apps/infra_flask_app') {
                sh "docker build -t infra_flask_image:${env.INFRA_FLASK_VERSION} ."
                sh "docker run -it --name infra_flask -p 5000:5000 -d infra_flask_image:${env.INFRA_FLASK_VERSION}"
            }
        }
        }

        stage('build and test flask_app') {
            steps {
                dir('/var/lib/jenkins/workspace/deployment/final_project/apps/flask_app') {
                sh "docker build -t flask_app_image:${env.FLASK_APP_VERSION} ."
                sh "docker run -it --name flask_app -p 5001:5001 -d flask_app_image:${env.FLASK_APP_VERSION}"
            }
        }
    }

        stage('push to dockerhub infra_app') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                    sh "docker tag infra_flask_image:${env.INFRA_FLASK_VERSION} sivanmarom/infra_flask:${env.INFRA_FLASK_VERSION}"
                    sh "docker push sivanmarom/infra_flask:${env.INFRA_FLASK_VERSION}"
                }
            }
        }

        stage('push to dockerhub flask_app') {
            steps {
                sh "docker tag flask_app_image:${env.FLASK_APP_VERSION} sivanmarom/flask_app:${env.FLASK_APP_VERSION}"
                sh "docker push sivanmarom/flask_app:${env.FLASK_APP_VERSION}"
            }
        }

        stage('create EKS cluster') {
            steps {
                dir('/var/lib/jenkins/workspace/deployment/final_project/terraform/eks') {
                sh 'terraform init'
                sh 'terraform apply --auto-approve'
            }
            }
        }

        stage('apps deploy') {
            steps {
                dir('/var/lib/jenkins/workspace/deployment/final_project/k8s') {
                script {
                    def imageTag_flask = env.FLASK_APP_VERSION
                    def imageTag_infra = env.INFRA_FLASK_VERSION
                    sh "sed -i 's|{{IMAGE_TAG}}|${imageTag_infra}|' infra-flask-deployment.yaml"
                    sh "sed -i 's|{{IMAGE_TAG}}|${imageTag_flask}|' flask-app-deployment.yaml"
                    sh 'kubectl apply -f infra-flask-deployment.yaml'
                    sh 'kubectl apply -f flask-app-deployment.yaml'
                    sh 'kubectl get all --namespace flask-space'
                }
            }
        }
        }
        stage('update versions') {
            steps {
                script {
                    def increment = 0.01
                    def infraVersion = Double.parseDouble(env.INFRA_FLASK_VERSION) + increment
                    def flaskVersion = Double.parseDouble(env.FLASK_APP_VERSION) + increment
                    env.INFRA_FLASK_VERSION = String.format('%.2f', infraVersion)
                    env.FLASK_APP_VERSION = String.format('%.2f', flaskVersion)
                }
            }
        }
    }
}