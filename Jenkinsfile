pipeline {
    agent any

    environment {
        TMPDIR = '/tmp'
        PATH = "$PATH:/home/ec2-user/bin"
    }

    stages {
        stage('get-build-pipeline') {

            steps {
              sh 'ssh-add /var/jenkins_home/.ssh/itsshared'
              dir('BuildRepo'){
                git url: 'git@gitlab.itsshared.net:aws/shared-services.git'
              }
              sh 'echo "--- Content of Build README ---"'
              sh 'ls BuildRepo/README.md'
              sh 'echo "-------------------------------"'
            }
        }
        stage('run tests') {
            steps {
            sh '''
                # Run All Tests :
                cd BuildRepo
                ls
                rm -rf ./venv
                python3.6 -m venv ./venv
                . ./venv/bin/activate
                pip install -r requirements.txt
                fab terraform.test_all
               '''
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
                rm -rf ./venv
                python3.6 -m venv ./venv
                . ./venv/bin/activate
                pip install -r requirements.txt
                fab terraform.test_all
               '''
    }

    private def deployRpms(){
            sh '''
                # Deploy RPMs :
                rm -rf ./venv
                echo ${ARCHNO}
                python3.6 -m venv ./venv
                source ./venv/bin/activate
                pip install -r requirements.txt
                fab deploy_rpms:env=test
                fab deploy_rpms:env=stagenp
                fab deploy_rpms:env=stagepr
               '''
    }

    private def deployToEnvironment(environment){
            println "Version of the artifact tar : ${env.ARCHNO}"

            withEnv(["ENVIRONMENT=${environment}"]) {
                sh '''
                    # Deploy to Environment :
                    mkdir -p ${ENVIRONMENT}
                    pushd ${ENVIRONMENT}
                    aws s3 cp s3://dwp-ios-admin-artifacts/${JOB_NAME}-${ARCHNO}.tar ${JOB_NAME}-${ARCHNO}.tar
                    tar -xvf ${JOB_NAME}-${ARCHNO}.tar
                    rm ${JOB_NAME}-${ARCHNO}.tar
                    python3.6 -m venv ./venv
                    source ./venv/bin/activate
                    pip install -r requirements.txt
                    fab deploy:env=${ENVIRONMENT}
                    popd ${ENVIRONMENT}
                '''
            }
        }
