pipeline {
    agent any

    environment {
        GITHUB_REPO = 'swapnildtech/tomcat' // Your repository
        GITHUB_SHA = '' // Initialize, will set in stages
    }

    stages {
        stage('Validate GitHub Token') {
            steps {
                withCredentials([string(credentialsId: 'jenkin-personal1', variable: 'GITHUB_TOKEN')]) {
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
                }
            }
        }

        stage('Testing') {
            steps {
                script {
                    echo 'Running tests...'
                    def testResult = sh(script: 'echo "Test passed!"', returnStdout: true).trim()
                    echo "Test Result: ${testResult}"
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    echo 'Build Started'
                    GITHUB_SHA = env.GIT_COMMIT ?: sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    echo "Using GITHUB_SHA: ${GITHUB_SHA}"
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
                    sh 'echo "Application deployed!"'
                }
            }
        }
    }

    post {
        success {
            script {
                echo "Commit SHA is: ${GITHUB_SHA}"
                updateGitHubStatus('success')
            }
        }
        failure {
            script {
                echo "Commit SHA is: ${GITHUB_SHA}"
                updateGitHubStatus('failure')
            }
        }
        unstable {
            script {
                echo "Commit SHA is: ${GITHUB_SHA}"
                updateGitHubStatus('error')
            }
        }
    }
}

def updateGitHubStatus(String status) {
    withCredentials([string(credentialsId: 'jenkin-personal1', variable: 'GITHUB_TOKEN')]) {
        echo "Updating GitHub status to '${status}' for commit SHA '${GITHUB_SHA}'"
        def response = sh(script: """
            curl -X POST -H "Authorization: token \$GITHUB_TOKEN" \
            -H "Accept: application/vnd.github.v3+json" \
            -d '{"state": "${status}", "context": "continuous-integration/jenkins"}' \
            "https://api.github.com/repos/${GITHUB_REPO}/statuses/${GITHUB_SHA}"
        """, returnStdout: true).trim()
        
        echo "GitHub response: ${response}"

        if (response.contains("Resource not accessible by personal access token")) {
            error "Failed to update status: ${response}"
        }
    }
}
