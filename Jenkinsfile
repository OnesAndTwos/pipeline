pipeline {

    agent none

    stages {

        stage('hello') {
            agent any
            steps {
                sh 'echo Hello'
            }

        }

        stage('Can I Deploy?') {
            agent none
            steps {
                milestone(ordinal: 1, label: "BOB")
                script {
                    timeout(time: 20, unit: 'DAYS') {
                        input 'Deploy to stage.'
                    }
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