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
                echo 'Running basic tests (e.g., HTML linting/validation placeholder)...'
                // For a real project, you would integrate actual testing frameworks here:
                 sh 'pip3 install --user html5validator'       // installs under /home/jenkins/.local/bin
    sh 'export PATH=$HOME/.local/bin:$PATH'       // make sure Jenkins sees the binary
    sh 'html5validator --root build/'   
                // - Basic content checks
                // - If JavaScript, unit tests (e.g., 'npm test')
                // For this tutorial, we'll use a placeholder.
                script {
                    sh 'echo "Simulating tests... All tests passed!"'
                    // Example of a simple check (e.g., verify index.html exists)
                    sh '[ -f build/index.html ] || { echo "ERROR: index.html not found!"; exit 1; }'
                }
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