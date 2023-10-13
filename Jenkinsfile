pipeline {
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }
    agent any
    environment {
        AWS_REGION = 'eu-central-1'  // Set your AWS region
        AWS_ACCOUNT_ID = "908177614064"
        AWS_EB_ENV_NAME = 'Jenkins-app-env'  // Set your Elastic Beanstalk environment name
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
                sh 'zip -r my-app.zip .'
            }
        }
        stage('Upload to S3') {
            steps {
                // Upload the zipped code to an S3 bucket
                withCredentials([usernamePassword(credentialsId: 'aws_eb_access', accessKeyVariable: 'AWS_ACCESS_KEY', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh 'aws s3 cp my-app.zip s3:/elasticbeanstalk-eu-central-1-908177614064/'
                }
            }
        }
        stage('Deploy to Elastic Beanstalk') {
            steps {
                script {
                    withAWS(region: env.AWS_REGION, roleAccount: env.AWS_ACCOUNT_ID) {
                        sh '''
                            aws elasticbeanstalk create-application-version --application-name jenkins-app \
                            --version-label Jenkins-${BUILD_ID} --source-bundle S3Bucket=elasticbeanstalk-eu-central-1-908177614064,S3Key=my-app.zip

                            aws elasticbeanstalk update-environment --environment-name $AWS_EB_ENV_NAME --version-label Jenkins-${BUILD_ID}
                        '''
                    }
                }
            }
        }
    }
}
