pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'docker build -t devops-node-project .'
            }
        }
    }
}
