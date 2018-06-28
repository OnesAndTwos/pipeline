def didTimeout = false
pipeline {

    agent none
    stages {

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
                script {
                    try {
                        timeout(time: 20, unit: 'DAYS') {
                            input 'Deploy to stage.'
                        }
                    }
                    catch (org.jenkinsci.plugins.workflow.steps.FlowInterruptedException e) {
                        didTimeout = true
                        print e
                    }
                }
             }
        }

        stage('nested stage') {
            when {
                expression { !didTimeout }
            }
            stages {
                stage('hello again') {
                    agent any
                    steps {
                        milestone(ordinal: 2, label: "BUILD_START_MILESTONE")
                        sh 'echo Hello'
                    }

                }

                stage('hello again again') {
                    agent any
                    steps {
                        milestone(ordinal: 3, label: "BUILD_START_MILESTONE")
                        sh 'echo Hello'
                    }
                 }
            }
        }

    }

    post {
        aborted {
            script {
                currentBuild.result = 'SUCCESS'
            }
        }
        always {
            script {
                if (didTimeout) {
                     currentBuild.result = 'SUCCESS'
                }
            }
        }
    }
}