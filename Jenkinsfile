pipeline {
    agent any

    tools {nodejs "node"}
    environment {
      SEMGREP_APP_TOKEN = credentials('semgrep-scan')
    //   DOJO_HOST = 'http://localhost:9000'
    //   DOJO_API_TOKEN = 'd56d4b30c4b8e877dc0a53fcd46994973f547e68'
    }

    stages {

        stage('Install Dependencies') {
            steps {
                echo 'Installing dependencies...'
                // sh 'rm -r node_modules'
                sh 'npm install'
            }
        }

        stage('Semgrep Scan') {
            steps {
                sh '''
                docker run \
                -e SEMGREP_APP_TOKEN=$SEMGREP_APP_TOKEN \
                -v "$(pwd):$(pwd)" --workdir $(pwd) \
                returntocorp/semgrep semgrep ci --code --junit-xml-output semgrep-report.xml'''
                sh 'exit 0' //continue build otherwise use delete exit code
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'semgrep-report.xml', skipPublishingChecks: true
                }
            }
        }

        // Secret scannning
        stage ('Check secrets') {
            steps {
                sh 'docker run  gesellix/trufflehog --json https://github.com/shubnimkar/CI_CD_Devsecops.git > trufflehog.json'

                script {
                    def jsonReport = readFile('trufflehog.json')
                    
                    def htmlReport = """
                    <html>
                    <head>
                        <title>Trufflehog Scan Report</title>
                    </head>
                    <body>
                        <h1>Trufflehog Scan Report</h1>
                        <pre>${jsonReport}</pre>
                    </body>
                    </html>
                    """
                    
                    writeFile file: 'scanresults/trufflehog-report.html', text: htmlReport
                }
                
                archiveArtifacts artifacts: 'scanresults/trufflehog-report.html', allowEmptyArchive: true
            }
        }

        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'  
            }
             sh 'exit 0' 
        }
    
    
        stage('Snyk Scan') {
            steps {
                echo 'Snyk Scanning...'
                snykSecurity(
                    snykInstallation: 'Snyk-Scan',
                    snykTokenId: 'Snyk-Scan',
                    severity: 'low',
                    failOnIssues: 'false'
                )
                
            }
        }

         stage('Deploy to DEV') {
            steps {
                echo 'Deploying...'
                echo 'DEPLOY TO DEV WAS SUCCESSFUL!!!'
            }
        }
        

       stage('Dastardly Scan') {
            steps {
                echo 'Launch app...'
                    sh 'docker-compose up --detach'
                echo 'Dastardly Scanning...'
                    sh '''
                    docker run --network host --user $(id -u) -v ${WORKSPACE}:${WORKSPACE}:rw \
                    -e BURP_START_URL=http://localhost:4000 \
                    -e BURP_REPORT_FILE_PATH=${WORKSPACE}/dastardly-report.xml \
                    public.ecr.aws/portswigger/dastardly:latest \
                    '''
                    sh 'exit 0'
                
                echo 'Dastardly Scanning Completed.'
                // echo 'Upload Dastardly Scan to DefectDojo'
                // steps {
                //     sh '''
                //     upload-results.py --host $DOJO_HOST --api_key $DOJO_API_TOKEN \
                //     --engagement_id 1 --product_id 1 --lead_id 1 --environment "Production" \
                //      --result_file dastardly-report.xml --scanner "Snyk Scan"
                //     '''
                // }
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'dastardly-report.xml', skipPublishingChecks: true
                }
            }
        }

       
        stage('Deploy to PROD') {
            steps {
                echo 'Deploying...'
                echo 'DEPLOY TO PROD WAS SUCCESSFUL!!!'
            }
        }

    }

    
}