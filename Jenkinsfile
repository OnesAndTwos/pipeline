pipeline {

    agent any

    stages {

        stage('hello') {
            steps {
                sh 'echo Hello'
            }

        }

        stage('hello again') {
            input {
                        message "Should we continue?"
                        ok "Yes, we should."
                    }

            steps {
                sh 'echo Hello'
            }

        }

    }

    post {
        always {
            deleteDir()
        }
    }
}