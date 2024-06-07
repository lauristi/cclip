pipeline {
    agent any

    environment {
        LOG_FILE = "pipeline.log"
        GIT_REPO = 'github.com/lauristi/cclip.git'
        BRANCH = 'master'
        DEPLOY_PATH = '/var/www/app/ServerProjects/phyton'
    }

    stages {
        stage('clean') {
            steps {
                script {
                    // Limpar diretório de artefatos antigos
                    sh "rm -rf ${env.DEPLOY_PATH}/*"
                }
            }
        }

        stage('01- Checkout') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'JENKINS_TOKEN', variable: 'GITHUB_TOKEN')]) {
                        try {
                            sh """
                                git clone https://${GITHUB_TOKEN}@${GIT_REPO}
                                cd cclip
                                git checkout ${BRANCH}
                            """ 
                        } catch (Exception e) {
                            TratarErro(e)
                        }
                    }
                }
            }
        }

        stage('02- Deploy on server') {
            steps {
                script {
                    try {
                        sh """
                            sudo mkdir -p "${env.DEPLOY_PATH}"
                            sudo cp -r cclip.py "${env.DEPLOY_PATH}/" && echo "Copy succeeded" || echo "Copy failed"
                            sudo chown -R www-data:www-data "${env.DEPLOY_PATH}/" && echo "Chown succeeded" || echo "Chown failed"
                        """
                    } catch (Exception e) {
                        TratarErro(e)
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                archiveArtifacts artifacts: "${env.LOG_FILE}", allowEmptyArchive: true
                cleanWs()
            }
        }
    }
}

def TratarErro(Exception e) {
    currentBuild.result = 'FAILURE'
    echo "--------------------------------------------------------------"
    echo "Deploy failed: ${e.message}"
    echo "--------------------------------------------------------------"
    error('Deploy failed')
    echo "--------------------------------------------------------------"
}
