pipeline {

    agent any
    tools {
        maven 'Maven 3.9.6'
    }

    stages {

        stage ('Build') {
            steps {
                sh 'echo === build stage Java app with Maven ==='
		sh 'mvn --version'
		sh 'cd old_depproject; mvn -Dmaven.test.failure.ignore=true -DskipTests clean install'

            }
        }

        stage('Semgrep-Scan') {
            steps {
                sh '''docker pull returntocorp/semgrep && \
                docker run \
                -e SEMGREP_APP_TOKEN=$SEMGREP_APP_TOKEN \
                -v "$(pwd):$(pwd)" --workdir $(pwd) \
                returntocorp/semgrep semgrep ci '''
                sh 'exit 0'
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'semgrep-report.xml', skipPublishingChecks: true
                }
            }
        }
    
        stage('Snyk') {
            steps {
                echo 'Snyk Scanning...'
                snykSecurity(
                    snykInstallation: 'Snyk-Scan',
                    snykTokenId: 'Snyk-Scan',
                    severity: 'low',
                    failOnIssues: 'false'
                )
                sh 'exit 0' 
            }
        }
        

        stage('Dastadrly Scan...') {
            steps {
                echo 'Dastardly Scanning..'
                    sh 'docker pull public.ecr.aws/portswigger/dastardly:latest'
                    cleanWs()
                    sh '''
                    docker run --user $(id -u) -v ${WORKSPACE}:${WORKSPACE}:rw \
                    -e BURP_START_URL=https://juice-shop.herokuapp.com \
                    -e BURP_REPORT_FILE_PATH=${WORKSPACE}/dastardly-report.xml \
                    public.ecr.aws/portswigger/dastardly:latest \
                    '''
                    sh 'exit 0'
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'dastardly-report.xml', skipPublishingChecks: true
                }
            }
        }

        stage('PROD') {
            steps {
                echo 'Deploying....'
            }
        }


    }
}

