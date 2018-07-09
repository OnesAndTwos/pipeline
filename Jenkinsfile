pipeline {
    agent any

    environment {
        TMPDIR = '/tmp'
        PATH = "$PATH:/home/ec2-user/bin"
    }

    stages {
        stage('get-build-pipeline') {
            steps {
              dir('BuildRepo'){
                git url: 'https://gitlab.itsshared.net/ryan.dutton/jenkins-ci-pipeline.git'
              }
              sh 'echo "--- Content of Build README ---"'
              sh 'cat BuildRepo/README.md'
              sh 'echo "-------------------------------"'
            }
        }
        stage('run tests') {
            steps {
                runTests()
            }
        }

        stage('upload artifact') {
            steps {
                script {
                    def archno = sh(script:'git describe', returnStdout: true)
                    env.ARCHNO = archno.trim()
                    println "${env.ARCHNO}"
                }

                deployRpms()

                sh '''
                    # Create and Upload Artifact :
                    tar -cvf ${JOB_NAME}-${ARCHNO}.tar --exclude='.terraform' \
                     --exclude='./puppet/vendor' --exclude='./puppet/.tmp' --exclude='./venv' .
                    aws s3 cp ${JOB_NAME}-${ARCHNO}.tar s3://dwp-ios-admin-artifacts/ --acl bucket-owner-read
                   '''
            }
            post {
                always{
                    deleteDir()
                }
            }
        }

        stage('deploy to test') {
            steps {
                deployToEnvironment('test')
            }
        }

        stage('approval') {
            agent none
            steps {
                timeout(time: 2, unit: 'MINUTES'){
                    input 'Deploy to stage.'
                }
            }
        }

        stage('deploy to stagenp') {
            steps {
                deployToEnvironment('stagenp')
            }
        }

        stage('deploy to stagepr') {
            steps {
                deployToEnvironment('stagepr')
            }
        }
    }

    post {
        always{
            deleteDir()
        }
    }
}

    private runTests() {
            sh '''
                # Run All Tests :
                pwd
                rm -rf ./venv
                python3.6 -m venv ./venv
                . ./venv/bin/activate
                eval $(ssh-agent)
                pip install -r requirements.txt
                fab terraform.test_all
               '''
        }
    }

    private def deployRpms(){
        withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'cloud_services_cd',  \
                                   keyFileVariable: 'KEY_PATH')]) {

            sh '''
                # Deploy RPMs :
                rm -rf ./venv
                echo ${ARCHNO}
                eval $(ssh-agent)
                ssh-add ${KEY_PATH}
                python3.6 -m venv ./venv
                source ./venv/bin/activate
                pip install -r requirements.txt
                fab deploy_rpms:env=test
                fab deploy_rpms:env=stagenp
                fab deploy_rpms:env=stagepr
                ssh-agent -k
               '''
        }
    }

    private def deployToEnvironment(environment){
        withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'cloud_services_cd',  \
                                               keyFileVariable: 'KEY_PATH')]) {
            println "Version of the artifact tar : ${env.ARCHNO}"

            withEnv(["ENVIRONMENT=${environment}"]) {
                sh '''
                    # Deploy to Environment :
                    mkdir -p ${ENVIRONMENT}
                    pushd ${ENVIRONMENT}
                    aws s3 cp s3://dwp-ios-admin-artifacts/${JOB_NAME}-${ARCHNO}.tar ${JOB_NAME}-${ARCHNO}.tar
                    tar -xvf ${JOB_NAME}-${ARCHNO}.tar
                    rm ${JOB_NAME}-${ARCHNO}.tar
                    eval $(ssh-agent)
                    ssh-add ${KEY_PATH}
                    python3.6 -m venv ./venv
                    source ./venv/bin/activate
                    pip install -r requirements.txt
                    fab deploy:env=${ENVIRONMENT}
                    popd ${ENVIRONMENT}
                    ssh-agent -k
                '''
            }
        }
    }
