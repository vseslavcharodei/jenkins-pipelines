pipeline {
    agent any

    parameters {
        choice(name: 'APP_VERSION', choices: getAvailableVersions(), description: 'Select the version to build (Git branch name)')
    }

    environment {
        APP_NAME = "myapp"
        GIT_REPO = "${env.GIT_REPO}"
        WORKSPACE_DIR = "${env.WORKSPACE}"
    }

    stages {
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

        stage('Build DEB') {
            steps {
                sh '''
                echo "Building DEB package for ${APP_NAME} version ${params.APP_VERSION}"
                mkdir -p ${WORKSPACE_DIR}/${APP_NAME}/DEBIAN
                echo "Package: ${APP_NAME}" > ${WORKSPACE_DIR}/${APP_NAME}/DEBIAN/control
                echo "Version: ${params.APP_VERSION}" >> ${WORKSPACE_DIR}/${APP_NAME}/DEBIAN/control
                echo "Depends: coreutils" >> ${WORKSPACE_DIR}/${APP_NAME}/DEBIAN/control
                dpkg-deb --build ${WORKSPACE_DIR}/${APP_NAME}
                '''
            }
        }

        stage('Archive Packages') {
            steps {
                archiveArtifacts artifacts: '**/*.rpm, **/*.deb', fingerprint: true
            }
        }
    }
}

def getAvailableVersions() {
    def gitRepo = "${env.GIT_REPO}"
    def branches = sh(script: "git ls-remote --heads ${gitRepo} | awk -F/ '{print \$NF}'", returnStdout: true).trim().split("\n")
    return branches.join("\n")
}
