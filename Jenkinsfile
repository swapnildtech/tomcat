pipeline {
    agent any
    environment {
        GITHUB_REPO = 'swapnildtech/tomcat'  // Repository identifier
        GITHUB_SHA = ''                       // Placeholder for commit SHA
    }
    stages {
        stage('Checkout') {
            steps {
                // Checkout the repository using SSH
                git branch: 'main', credentialsId: 'github-ssh-key', url: 'git@github.com:swapnildtech/tomcat.git'
            }
        }
        stage('Validate SSH Key Connectivity') {
            steps {
                script {
                    echo 'Validating SSH key connectivity...'
                    // Test SSH connection
                    def sshTest = sh(script: 'ssh -T git@github.com', returnStatus: true)
                    if (sshTest != 0) {
                        error 'SSH key validation failed. Please check your SSH setup.'
                    } else {
                        echo 'SSH key connectivity validated successfully.'
                    }
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    echo 'Build Started'
                    // Get the current commit SHA
                    GITHUB_SHA = env.GIT_COMMIT ?: sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    echo "Using GITHUB_SHA: ${GITHUB_SHA}"
                    sh 'echo "Building application..."'
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

// Function to update GitHub status
def updateGitHubStatus(String status) {
    withCredentials([string(credentialsId: 'jenkin-personal1', variable: 'GITHUB_TOKEN')]) {
        echo "Updating GitHub status to '${status}' for commit SHA '${GITHUB_SHA}'"
        def response = sh(script: """
            curl -s -o response.json -w "%{http_code}" -X POST -H "Authorization: token \$GITHUB_TOKEN" \
            -H "Accept: application/vnd.github.v3+json" \
            -d '{"state": "${status}", "context": "continuous-integration/jenkins"}' \
            "https://api.github.com/repos/${GITHUB_REPO}/statuses/${GITHUB_SHA}"
        """, returnStdout: true).trim()

        // Read the response code
        def responseCode = response.tokenize().last()
        def responseBody = readFile('response.json')

        echo "GitHub response: ${responseBody}"

        if (responseCode != '200') {
            error "Failed to update status: HTTP ${responseCode}, Response: ${responseBody}"
        }
    }
}
