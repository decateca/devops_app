pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                // Pull latest code from GitHub (triggered by webhook)
                checkout scm
            }
        }
        stage('Build & Test') {
            steps {
                // Run your build and test logic
                sh 'echo "Building and testing..."'
            }
        }
        stage('Package/Publish') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentails-ptower']]){
                    sh 'zip -r myapp.zip .'
                    sh 'aws s3 cp myapp.zip s3://cicd-dev-deca/b001_pro_search/compile/myapp.zip'
                }
            }
        }
        /*stage('Update Infrastructure') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentails-ptower']]){
                    // Use Terraform or AWS CLI to update the ASG's launch template
                    // or user data with the new artifact details.
                    sh 'terraform init'
                    sh 'terraform plan -out=tfplan'
                    sh 'terraform apply -auto-approve tfplan'
                }                
            }
        }*/
        stage('Refresh Instances') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentails-ptower']]) {
                    sh '''
                    set +e
                    aws autoscaling start-instance-refresh --auto-scaling-group-name b001-pro-search-dev-asg --preferences '{"MinHealthyPercentage":90, "InstanceWarmup":300}'
                    RET_CODE=$?
                    if [ $RET_CODE -eq 254 ]; then
                        echo "Instance Refresh already in progress. Skipping new refresh trigger."
                    elif [ $RET_CODE -ne 0 ]; then
                        echo "Error starting Instance Refresh, exit code $RET_CODE."
                        exit $RET_CODE
                    fi
                    set -e
                    '''
                }
            }
        }
    }
    post {
        success {
            echo 'Deployment completed successfully!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}
