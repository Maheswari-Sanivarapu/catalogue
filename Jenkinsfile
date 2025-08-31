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
        stage('unit tests'){
            steps{
                script{
                    sh """
                        echo "unit tests"
                    """
                }
            }
        }
        /* stage('Sonar scan'){
            environment{
                // Set the path to the SonarScanner executable
                // The 'tool' directive provides the path based on the configured tool
                scannerHome = tool 'sonar-7.2'
            }
            steps{
                script{
                    withSonarQubeEnv(installationName: 'sonar-7.2'){
                         // installationName: is the name of your SonarQube server configuration in Jenkins System Configuration
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        } */
        // enabling webhook in sonarqube server and wait for results
        /* stage('Quality Gate'){
            steps{
                timeout(time: 1, unit: 'HOURS'){
                waitForQualityGate abortPipeline: true }
            }
        } */
        /* stage('Check Dependabot Alerts') {
            environment{
                GITHUB_TOKEN = credentials('github-token')
            }
            steps {
                script {
                        // Fetch the alerts JSON and save to file
                    def response = sh(
                        script: """
                            curl -s -H "Accept: application/vnd.github+json" \
                                 -H "Authorization: token ${GITHUB_TOKEN}" \
                                 https://api.github.com/repos/Maheswari-Sanivarapu/catalogue/dependabot/alerts
                        """,
                        returnStdout: true
                    ).trim()
                        // Parse and evaluate the JSON in Groovy
                    def json = readJSON text: response

                    def highOrCritical = alerts.findAll { alert ->
                        def severity = alert?.security_advisory?.severity?.toLowerCase()
                        def state = alert?.state?.toLowerCase()
                        return (severity == 'high' || severity == 'critical')
                    }

                    if (highOrCritical.size() > 0) {
                        echo "❌ Found ${highOrCritical.size()} high/critical Dependabot alerts!"
                        error("Failing build due to high/critical vulnerabilities.")
                    } else {
                        echo "✅ No high or critical Dependabot alerts found."
                    }
                }
            }
        }    */
        stage('Build the docker image '){
            steps{
                script{
                    withAWS(credentials: 'aws-creds', region: 'us-east-1') { // here this line will be executed only within this stage
                        //  ECR Registry is created manually and here building and pushing the image to ECR which is a private repository
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
            // here it will trigger the cd and here catalogue-cd is downstream to catalogue-ci
            // here keeping when condition so everytime it won't trigger the CD,when developer wrote the code and it is scanned successfully then only CD will be triggered or else it won't be triggered so keeping when condition
            when {
                expression { params.deploy } // here it will the value from parameters
            }
            steps{
                script{
                    build job: 'catalogue-cd', // this is the downstream job
                    // here we are the building the downstream job with parameters
                    parameters: [
                        string(name: 'appVersion', value: "${appVersion}"), // appVersion of the CI is giving it to CD and string(name: 'appVersion') name we got from catalogue-cd parameters and we are passing the appVersion of CI i.e.from readPackageJson file to CD
                        string(name: 'deploy_to', value: 'dev') // here string(name: 'deploy_to') is also parameter name from catalogue-cd
                    ],
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