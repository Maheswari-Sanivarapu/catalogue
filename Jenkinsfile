pipeline{
    agent {
        label 'AGENT_1'
    }
    environment {
        appVersion = ''
        REGION = "us-east-1"
        ACC_ID = "557690617909"
        PROJECT = "roboshop"
        COMPONENT = "catalogue"
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }
    parameters {
        booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value' )
    }
    stages{
        stage('Read package.json file') {
            steps{
                script{
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "Package version : ${appVersion}"
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
        stage('Build the docker image '){
            steps{
                script{
                    withAWS(credentials: 'aws-creds', region: 'us-east-1') { // here this line will be executed only within this stage
                        sh """
                        aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com
                        docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} .
                        docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                        """
                    }
                }
            }
        }
        stage('Trigger catalogue-cd'){
            steps{
                script{
                    build job: 'catalogue-cd',
                    parameters: [
                        string(name: 'appVersion', value: '${appVersion}'),
                        string(name:'deploy_to', value: 'dev')
                    ]
                    propagate: false,
                    wait: false
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