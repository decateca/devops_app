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
        stage('Update Infrastructure') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentails-ptower']]){
                    // Use Terraform or AWS CLI to update the ASG's launch template
                    // or user data with the new artifact details.
                    sh 'terraform init'
                    sh 'terraform plan -out=tfplan'
                    sh 'terraform apply -auto-approve tfplan'
                }                
            }
        }
        stage('Refresh Instances') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentails-ptower']]){
                    // Trigger an instance refresh so existing instances pick up changes
                    sh '''
                    aws autoscaling start-instance-refresh \
                    --auto-scaling-group-name b001-pro-search-dev-asg \
                    --preferences '{"MinHealthyPercentage":90, "InstanceWarmup":300}'
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
