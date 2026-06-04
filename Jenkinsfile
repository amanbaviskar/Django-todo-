pipeline {
agent any

environment {
    IMAGE_NAME = "amanbaviskar477/django_todo"
    IMAGE_TAG = "${BUILD_NUMBER}"
    AWS_REGION = "ap-south-1"
    EKS_CLUSTER = "demo-cluster"
    HELM_RELEASE = "django-todo"
    HELM_CHART_PATH = "./django-todo-chart"
}

stages {

    stage('Checkout') {
        steps {
            checkout scm
        }
    }

    stage('Build Docker Image') {
        steps {
            sh '''
            docker build -t $IMAGE_NAME:$IMAGE_TAG .
            docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest
            '''
        }
    }

    stage('Push Docker Image') {
        steps {
            withCredentials([
                usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )
            ]) {
                sh '''
                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                docker push $IMAGE_NAME:$IMAGE_TAG
                docker push $IMAGE_NAME:latest

                docker logout
                '''
            }
        }
    }

    stage('Update Kubeconfig') {
        steps {
            sh '''
            aws eks update-kubeconfig --region $AWS_REGION --name $EKS_CLUSTER
            '''
        }
    }

    stage('Helm Deploy') {
        steps {
            sh '''
            helm upgrade --install $HELM_RELEASE $HELM_CHART_PATH \
              --set image.repository=$IMAGE_NAME \
              --set image.tag=$IMAGE_TAG
            '''
        }
    }

    stage('Verify Deployment') {
        steps {
            sh '''
            kubectl get pods
            kubectl get svc
            kubectl get ingress
            '''
        }
    }
}

post {
    success {
        echo 'Helm deployment to EKS completed successfully'
    }

    failure {
        echo 'Pipeline failed'
    }
}


}
