pipeline {
    agent any

    environment {
        APP_NAME = "myapp"
        GIT_REPO = "${env.GIT_REPO}"
        WORKSPACE_DIR = "${env.WORKSPACE}"
    }

    stages {
        stage('Initialize Parameters') {
            steps {
                script {
                    echo "Fetching available branches..."
                    
                    // Fetch available Git branches
                    def branches = sh(
                        script: "git ls-remote --heads ${env.GIT_REPO} | awk -F/ '{print \$NF}'",
                        returnStdout: true
                    ).trim().split("\n")

                    // Ensure at least one default branch exists
                    if (branches.isEmpty()) {
                        branches = ["main"]
                    }

                    echo "Available Versions: ${branches.join(', ')}"

                    // Update Jenkins job properties to register parameters
                    properties([
                        parameters([
                            choice(name: 'APP_VERSION', choices: branches, description: 'Select the version to build (Git branch name)')
                        ])
                    ])
                }
            }
        }

        stage('Checkout') {
            steps {
                script {
                    echo "Cloning repository from ${GIT_REPO} (Branch: ${params.APP_VERSION})"
                }
                git branch: "${params.APP_VERSION}", url: "${GIT_REPO}"
            }
        }

        stage('Build RPM') {
            steps {
                sh '''
                echo "Building RPM package for ${APP_NAME} version ${params.APP_VERSION}"
                tar czvf ${APP_NAME}-${params.APP_VERSION}.tar.gz ${APP_NAME}
                rpmbuild -ba ${APP_NAME}.spec --define "version ${params.APP_VERSION}"
                '''
            }
        }

        stage('Archive Packages') {
            steps {
                archiveArtifacts artifacts: '**/*.rpm', fingerprint: true
            }
        }
    }
}

