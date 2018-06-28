pipeline {

    stages {

        stage('hello') {
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