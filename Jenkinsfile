
pipeline {
    agent any
    stages {
        stage('hello') {
            steps { // steps executed on the Jenkins agent
                sh """
                    echo "hello from Github"
                """
            }
        }
    }
}