pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                dir('src/frontend') {
                    sh 'go build'
                }
            }
        }

    }

}
