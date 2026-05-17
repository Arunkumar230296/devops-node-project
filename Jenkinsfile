pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '283904064946'
        ECR_REPO = 'devops-node-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $ECR_REPO:$IMAGE_TAG .'
            }
        }

        stage('Login to ECR') {
            steps {
                sh 'aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com'
            }
        }

        stage('Push to ECR') {
            steps {
                sh 'docker tag $ECR_REPO:$IMAGE_TAG $IMAGE_URI'
                sh 'docker push $IMAGE_URI'
            }
        }

        stage('Update Helm Chart Image Tag') {
            steps {
                sh '''
                sed -i "s|image: .*|image: $IMAGE_URI|g" devops-node-chart/templates/deployment.yaml
                '''
            }
        }

        stage('Push Updated Helm Chart to GitHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                    sh '''
                    git config user.name "Jenkins"
                    git config user.email "jenkins@local"
                    git add devops-node-chart/templates/deployment.yaml
                    git commit -m "update helm image to build $BUILD_NUMBER" || true
                    git push https://$GIT_USER:$GIT_PASS@github.com/Arunkumar230296/devops-node-project.git HEAD:main
                    '''
                }
            }
        }
    }
}
