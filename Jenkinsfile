pipeline {
    agent any

    environment {
        APP_NAME = "myapp"
        GIT_REPO = "${env.GIT_REPO}"
        WORKSPACE_DIR = "${env.WORKSPACE}"
    }

    parameters {
        choice(name: 'APP_VERSION', choices: ['main'], description: 'Select the version to build (Git branch name)')
    }

    stages {
        stage('Setup Parameters') {
            steps {
                script {
                    // Fetch available branches
                    def branches = sh(script: "git ls-remote --heads ${env.GIT_REPO} | awk -F/ '{print \$NF}'", returnStdout: true).trim().replaceAll('\n', ',')
                    
                    // Apply dynamically fetched branches as choices
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
