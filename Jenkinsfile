pipeline {

    agent none

    stages {

        stage('hello') {
            agent any
            steps {
                milestone()
                sh 'echo Hello'
            }

        }

        stage("Can I Deploy?") {

            steps {
                milestone("")
                input {
                    message "Should we continue?"
                    ok "Yes, we should."
                }
            }

        }

        stage('hello again') {
            agent any
            steps {
                milestone(ordinal: 1, label: "BUILD_START_MILESTONE")
                sh 'echo Hello'
            }

        }

    }
}