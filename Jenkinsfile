pipeline {
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }
    agent any
    environment {
        AWS_REGION = 'eu-central-1'  // Set your AWS region
        AWS_ACCOUNT_ID = "908177614064"
        AWS_EB_ENV_NAME = 'Web-app-env'  // Set your Elastic Beanstalk environment name
    }
    triggers {
        GenericTrigger(
            causeString: 'Deployment Trigger',
            genericVariables: [
                    [key: 'COMMIT_MESSAGE', value: '$.head_commit.message'],
                    [key: 'TRIGGER_GIT_BRANCH', value: '$.ref'],
            ],
            printContributedVariables: true,
            printPostContent: printPostContent,
            regexpFilterText: '$COMMIT_MESSAGE',
            regexpFilterExpression: '^Releasing',
            tokenCredentialId: "elastik_beanstalk_github_token"
        )
    }
    stages {
        stage('Checkout SCM') {
            steps { 
                script {
                    git branch: 'main',
                        credentialsId: 'github-ssh-key',
                        url: 'git@github.com:bagrat92/jenkins-from-aws.git'
                }
            }

        }
        stage('Zip Application Code') {
            steps {
                // Zip your application code
                sh "zip -r web-test-Jenkins-${BUILD_ID}.zip ."
            }
        }
        stage('Upload to S3') {
            steps {
                // Upload the zipped code to an S3 bucket
                withAWS(credentials: 'aws_creds', region: env.AWS_REGION) {
                    sh '''
                        aws s3 cp web-test-Jenkins-${BUILD_ID}.zip s3://elasticbeanstalk-eu-central-1-908177614064/
                    '''
                }
            }
        }
        stage('Deploy to Elastic Beanstalk') {
            steps {
                script {
                    withAWS(credentials: 'aws_creds', region: env.AWS_REGION) {
                        sh '''
                            aws elasticbeanstalk create-application-version --application-name web-app \
                            --version-label Jenkins-${BUILD_ID} --source-bundle S3Bucket=elasticbeanstalk-eu-central-1-908177614064,S3Key=my-app.zip

                            aws elasticbeanstalk update-environment --environment-name $AWS_EB_ENV_NAME --version-label Jenkins-${BUILD_ID}
                        '''
                    }
                }
            }
        }
    }
}
