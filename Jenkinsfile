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
                milestone(ordinal: 1, label: "BUILD_START_MILESTONE")
            }

            steps {
                sh 'echo Hello'
            }

        }

    }
}