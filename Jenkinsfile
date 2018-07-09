def didTimeout = false
pipeline {
    agent none

    stages {
        stage('get build pipeline') {
            agent any
            triggers { cron('H */1 * * 1-5') }
            steps {
              dir('BuildRepo'){
                git url: 'https://github.com/example/build.git'
              }
            }
        }

        stage('hello') {
            agent any
            steps {
                sh 'echo Hello'
            }

        }

        stage('approval') {
             agent none
             steps {

                milestone(ordinal: 1, label: "BOB")

                timeout(time: 20, unit: 'DAYS') {
                    input 'Deploy to stage.'
                }

             }
        }

        stage('hello again') {
            agent any
            steps {
                milestone(ordinal: 2, label: "BUILD_START_MILESTONE")
                sh 'echo Hello me'
            }

        }

        stage('hello again again') {
            agent any
            steps {
                milestone(ordinal: 3, label: "BUILD_START_MILESTONE")
                sh 'echo Hello you'
            }
        }
    }
}
