pipeline {
    agent any

    environment {
        GITHUB_REPO = 'swapnildtech/tomcat' // Your repository
        GITHUB_SHA = '' // Initialize, will set in stages
    }

    stages {
        stage('Validate GitHub Token') {
            steps {
                withCredentials([string(credentialsId: 'jenkin-personal', variable: 'GITHUB_TOKEN')]) {
                    script {
                        def response = sh(script: """
                            curl -s -o /dev/null -w "%{http_code}" -H "Authorization: token \$GITHUB_TOKEN" \
                            "https://api.github.com/user"
                        """, returnStdout: true).trim()
                        
                        if (response != '200') {
                            error "GitHub token validation failed. HTTP Status: ${response}"
                        } else {
                            echo "GitHub token is valid."
                        }
                    }
                }
            }
        }

        stage('Preparation') {
            steps {
                script {
                    echo 'Preparing for the pipeline...'
                    echo "BRANCH NAME: ${env.BRANCH_NAME}"
                    echo sh(returnStdout: true, script: 'env')
                }
            }
        }

        stage('Testing') {
            steps {
                script {
                    echo 'Running tests...'
                    // Simulate a test command
                    def testResult = sh(script: 'echo "Test passed!"', returnStdout: true).trim()
                    echo "Test Result: ${testResult}"
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    echo 'Build Started'
                    // Capture the GIT_COMMIT SHA in this stage
                    GITHUB_SHA = env.GIT_COMMIT ?: sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    echo "Using GITHUB_SHA: ${GITHUB_SHA}"
                    // Simulate a build command
                    sh 'echo "Building application..."'
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                script {
                    echo 'Deploying Application'
                    // Simulate a deployment command
                    sh 'echo "Application deployed!"'
                }
            }
        }
    }

    post {
        success {
            script {
                echo "Commit SHA is: ${GITHUB_SHA}" // Ensure correct SHA is set
                updateGitHubStatus('success')
            }
        }
        failure {
            script {
                echo "Commit SHA is: ${GITHUB_SHA}" // Ensure correct SHA is set
                updateGitHubStatus('failure')
            }
        }
        unstable {
            script {
                echo "Commit SHA is: ${GITHUB_SHA}" // Ensure correct SHA is set
                updateGitHubStatus('error')
            }
        }
    }
}

def updateGitHubStatus(String status) {
    withCredentials([string(credentialsId: 'jenkin-personal1', variable: 'GITHUB_TOKEN')]) {
        echo "Updating GitHub status to '${status}' for commit SHA '${GITHUB_SHA}'"
        def response = sh(script: """
            curl -X POST -H "Accept: application/vnd.github+json" \
            -H "Authorization: token \$GITHUB_TOKEN" \
            -H "X-GitHub-Api-Version: 2022-11-28" \
            -d '{"state": "${status}", "context": "continuous-integration/jenkins"}' \
            "https://api.github.com/repos/${GITHUB_REPO}/statuses/${GITHUB_SHA}"
        """, returnStdout: true).trim()
        
        echo "GitHub response: ${response}"
    }
}
