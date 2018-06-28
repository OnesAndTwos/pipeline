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
                        status "COMPLETED"
                    }

            steps {
                sh 'echo Hello'
            }

        }

    }

    post {
         aborted {
              script {
                currentBuild.result = 'SUCCESS'
              }
            }
    }
}