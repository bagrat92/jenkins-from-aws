pipeline {
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }
    agent any
    environment {
        AWS_REGION = 'eu-central-1'  // Set your AWS region
        AWS_ACCOUNT_ID = "908177614064" // Your Account ID
        AWS_EB_ENV_NAME = 'Web-test-env'  // Set your Elastic Beanstalk environment name
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
        // stage('Assume Role') {
        //     steps {
        //         script {
        //             // Assume an AWS IAM role for this pipeline
        //             def assumedRole = awsRoleAssumer(
        //                 credentials: 'aws_eb_access',
        //                 roleArn: 'arn:aws:iam::908177614064:role/aws_eb_roles_for_jenkins'
        //             )
        //         }
        //     }
        // }
        stage('Zip Application Code') {
            steps {
                // Zip your application code
                sh "zip -r web-test-Jenkins-${BUILD_ID}.zip ."
            }
        }
        stage('Upload to S3') {
            steps {
                // Upload the zipped code to an S3 bucket
                withCredentials([usernamePassword(credentialsId: 'aws_key', accessKeyVariable: 'AWS_ACCESS_KEY', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh 'aws s3 cp web-test-Jenkins-${BUILD_ID}.zip s3:/elasticbeanstalk-eu-central-1-908177614064/'
                }
            }
        }
        stage('Deploy to Elastic Beanstalk') {
            steps {
                script {
                    withAWS(credentials: 'aws_key', region: env.AWS_REGION) {
                        sh '''
                            aws elasticbeanstalk create-application-version --application-name web-test \
                            --version-label Jenkins-${BUILD_ID} --source-bundle S3Bucket=elasticbeanstalk-eu-central-1-908177614064,S3Key=web-test-Jenkins-${BUILD_ID}.zip

                            aws elasticbeanstalk update-environment --environment-name $AWS_EB_ENV_NAME --version-label Jenkins-${BUILD_ID}
                        '''
                    }
                }
            }
        }
    }
}
