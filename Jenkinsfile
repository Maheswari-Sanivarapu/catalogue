pipeline{
    agent {
        label 'AGENT-1'
    }
    environment {
        appVersion = ''
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }
    stages{
        stage('Read package.json file') {
            steps{
                script{
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "Package version  : ${appVersion}"
                }
            }
        }
        stage('Install the dependencies') {
            steps{
                script{
                    sh """
                        npm install
                    """
                }
            }
        }
    }
    post {
        always {
            echo 'I will always say Hello again!'
            deleteDir()
        }
        success {
            echo 'Hello Success'
        }
        failure {
            echo 'Hello Failure'
        }
    }
}