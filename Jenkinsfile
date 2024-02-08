pipeline {
    agent any

    tools {nodejs "node"}
    environment {
      SEMGREP_APP_TOKEN = credentials('semgrep-scan')
      DOJO_HOST = 'http://localhost:9000'
      DOJO_API_TOKEN = 'd56d4b30c4b8e877dc0a53fcd46994973f547e68'
    }

    stages {

        stage('DEV') {
            steps {
                echo 'Building...'
                sh 'npm install --no-audit'
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
                
                // echo 'Dastardly Scanning Completed.'
                // echo 'Upload Dastardly Scan to DefectDojo'
                // steps {
                //     sh '''
                //     upload-results.py --host $DOJO_HOST --api_key $DOJO_API_TOKEN \
                //     --engagement_id 1 --product_id 1 --lead_id 1 --environment "Production" \
                 //     --result_file dastardly-report.xml --scanner "Snyk Scan"
                //     '''
                // }
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