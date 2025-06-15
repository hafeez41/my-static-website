pipeline {
    agent any // The pipeline can run on any available agent

    environment {
        // AWS credentials ID configured in Jenkins (Manage Jenkins -> Manage Credentials)
        // Ensure this ID matches the one you set up for your AWS credentials
        AWS_CREDENTIALS_ID = 'aws-s3-deploy-credentials'
        S3_BUCKET_NAME = 'your-unique-devops-static-website-bucket' // Must match your Terraform bucket name
        AWS_REGION = 'us-east-1' // Must match your Terraform region
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                echo 'Checking out source code from GitHub...'
                // Checkout the SCM. Uses the repository configured in the Jenkins job.
                git branch: 'main', url: 'https://github.com/hafeez41/my-static-website.git/'
                // Replace YOUR_GITHUB_USERNAME and my-static-website with your actual GitHub details
            }
        }

        stage('Build') {
  steps {
    echo 'Building static website (simple copy for static content)...'
    script {
      sh 'mkdir -p build'                               // ensure build/ exists
      sh 'cp -r website/. build/'                       // copy only your website assets
      sh 'ls -l build'                                  // verify contents
    }
  }
}


        stage('Test') {
  steps {
    echo 'Setting up a virtualenv and running html5validator…'
    sh '''
      # Install venv support if it’s not already present
      sudo apt-get update
      sudo apt-get install -y python3-venv

      # Create & activate the virtualenv
      python3 -m venv venv
      . venv/bin/activate

      # Install html5validator into the venv
      pip install html5validator

      # Run the validator against your build directory
      html5validator --root build/ || exit 1
    '''
  }
}

        stage('Deploy to S3') {
            steps {
                echo "Deploying to S3 bucket: ${env.S3_BUCKET_NAME} in region ${env.AWS_REGION}..."
                // Use the 'awscli' Jenkins plugin step (requires AWS Steps plugin)
                // Ensure your 'aws-s3-deploy-credentials' is set up in Jenkins
                withAWS(credentials: env.AWS_CREDENTIALS_ID, region: env.AWS_REGION) {
                    sh "aws s3 sync build/ s3://${env.S3_BUCKET_NAME}/ --delete"
                    // The --delete flag removes files from S3 that are no longer in your local build directory.
                    // This keeps your S3 bucket in sync with your source code.
                }
                echo 'Deployment complete!'
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
            // You might add notifications here, e.g., email or Slack
        }
    }
}