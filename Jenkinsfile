pipeline {
    agent any

    environment {
        CUSTODIAN_BIN = '/var/lib/jenkins/custodian/bin/custodian'
        MAILER_CONFIG_PATH = '/var/lib/jenkins/custodian/bin/c7n-mailer' // Relative path to the mailer.yml file within the repository
        MAILER_FILE = '/var/lib/jenkins/mailer.yml'
    }
    parameters {
        string(name: 'POLICY_FILE_NAME', defaultValue: '', description: 'Name of the policy file')
        string(name: 'AWS_ACCESS_KEY_ID', defaultValue: '', description: 'AWS Access Key')
        string(name: 'AWS_SECRET_ACCESS_KEY', defaultValue: '', description: 'AWS Secret Access Key')
        string(name: 'AWS_REGION', defaultValue: '', description: 'AWS Region')
        string(name: 'Count', defaultValue: '', description: 'Count of Resources')

    }  

    stages {
        stage('Checkout') {
            steps {
                // Checkout the code from your Git repository
                git branch: 'master', credentialsId: 'GIT_PAT', url: 'https://github.com/Soumya220/custodian-policies.git'
            }
        }

        stage('Fetch Policy File') {
            steps {
                script {
                    // Use the POLICY_FILE_NAME parameter in your pipeline steps
                    echo "Building job with policy file: ${params.POLICY_FILE_NAME}"
                    
                    // Modify this line to use the POLICY_FILE_NAME parameter in your build steps
                    // sh "some_command --policy ${params.POLICY_FILE_NAME}"
                }
            }
        }
        stage('Execute Policy File') {
            steps {
                script {
                    // Use the POLICY_FILE_NAME parameter in your pipeline steps
                    echo "Building job with policy file: ${params.POLICY_FILE_NAME}"
                    
                    // Obtain AWS credentials from the frontend (you need to modify this part)
                    def awsAccessKey = params.AWS_ACCESS_KEY_ID ?: ''
                    def awsSecretKey = params.AWS_SECRET_ACCESS_KEY ?: ''
                    def awsRegion = params.AWS_REGION ?: ''
        
                    // Ensure the custodian executable has the correct permissions
                    sh "chmod +x $CUSTODIAN_BIN"
                    // Execute Cloud Custodian with the policy file
                    // sh "$CUSTODIAN_BIN run --cache-period 0 --output-dir=. ${params.POLICY_FILE_NAME}"
                    sh "pwd"
                    sh "$CUSTODIAN_BIN run --cache-period 0 --output-dir s3://my-bucket-custodian/ ${params.POLICY_FILE_NAME}"
                    //echo "sh $CUSTODIAN_BIN run --cache-period 0 --output-dir=.  ${params.POLICY_FILE_NAME}"
                    sh "sleep 1m"
                    sh "$CUSTODIAN_BIN report --output-dir s3://my-bucket-custodian/ ${params.POLICY_FILE_NAME} > report.txt"
                    stash name: 'report', includes: 'report.txt'
                    sh "cat report.txt"
                    sh '''
                        count=$(cat report.txt | wc -l)
                        echo "Count: ${count}"
                        if [ "${count}" -gt 1 ]; then
                            echo "Resource is Not Compliant"
                        else 
                            echo "Resource is Compliant"
                        fi   
                       '''
                }
            }
        }

        stage('Send Email Notification') {
            steps {
                script {
                        // Send email using c7n-mailer with the mailer.yml configuration
                        sh "chmod +x $MAILER_CONFIG_PATH"
                        sh "$MAILER_CONFIG_PATH --run --config $MAILER_FILE"
                        // sh 'echo $MAILER_FILE '
                }
            }
        }
        stage('Generate Reports') {
            steps {
                script {
                        // Generate the HTML report (assuming 'report.html' is the report generated)
                        sh "echo '<h1>Build Success/Failure Report</h1>' > report.html"
                        sh "echo '<p>Build Status: ${currentBuild.currentResult}</p>' >> report.html"
                        sh "echo '<p>More details at: ${env.BUILD_URL}</p>' >> report.html"
                }
            }
        }
    }
            post {
                always {
                    unstash 'report'
                    archiveArtifacts artifacts: 'report.txt', allowEmptyArchive: true
                }
            }
}
