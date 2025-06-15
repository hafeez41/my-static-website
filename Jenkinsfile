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
                // For a static website, "build" typically means ensuring all files are present
                // and correctly structured. No compilation step is usually needed.
                // In a more complex scenario, this might involve npm build, webpack, etc.
                script {
                    sh 'mkdir -p build' // Create a directory for build artifacts
                    sh 'cp -r ./* build/' // Copy all files to the build directory
                    sh 'ls -l build' // List files in the build directory to verify
                }
            }
        }
       stage('Test') {
  steps {
    echo 'Running basic static-site checks…'
    sh '''
      # 1) Ensure index.html exists
      if [ ! -f build/index.html ]; then
        echo "❌ ERROR: build/index.html not found"
        exit 1
      fi

      # 2) Check for a DOCTYPE
      grep -q "<!DOCTYPE html>" build/index.html \
        || { echo "❌ ERROR: Missing <!DOCTYPE html>"; exit 1; }

      # 3) Check for a <title> tag
      grep -q "<title>.*</title>" build/index.html \
        || { echo "❌ ERROR: Missing <title> tag"; exit 1; }

      # 4) (Optional) Smoke-test the live S3 endpoint
      # curl must be installed on the agent for this
      STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
        http://${S3_BUCKET_NAME}.s3-website-${AWS_REGION}.amazonaws.com/index.html)
      if [ "$STATUS" -ne 200 ]; then
        echo "❌ ERROR: S3 site returned HTTP $STATUS"
        exit 1
      fi

      echo "✅ Basic tests passed!"
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