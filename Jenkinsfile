node {

    // Ensure required environment variables are set
    if (!env.GIT_REPO) {
        error("ERROR: GIT_REPO environment variable is not set!")
    }

    // Define GIT_REPO local variable to pass value from env.GIT_REPO to ScriptlerScript
    def gitRepo = env.GIT_REPO
    
    // Define Job Properties and Parameters
    properties([
        parameters([
            [$class: 'CascadeChoiceParameter',
                name: 'APP_VERSION',
                script: [
                    $class: 'ScriptlerScript',
                    scriptlerScriptId: 'getGitBranches.groovy',
                    parameters: [
                        [name: 'gitRepo', value: gitRepo]
                    ]
                ],
                choiceType: 'PT_SINGLE_SELECT',
                description: 'Select an application version to build'
            ],
            choice(name: 'PACKAGE_TYPE', choices: ['rpm', 'deb', 'all'], description: 'Select package type to build')
        ])
    ])

    // Define other params to use in stages
    def appVersion = params.APP_VERSION
    def packageType = params.PACKAGE_TYPE
    def appName = gitRepo.tokenize('/').last().replaceAll(/\.git$/, '')
    def checkoutDir = "${appName}-${appVersion}"

    // Checkout Code from Git
    stage("Checkout") {
        script {
            echo "Cloning repository from ${gitRepo} (Branch: ${appVersion}) into directory ${checkoutDir}"

            // Checkout inside a subdirectory with the application name
            dir(checkoutDir) {
                git branch: "${appVersion}", url: "${gitRepo}"
            }

            echo "DEBUG: Listing files in ${checkoutDir} after checkout:"
            sh "ls -lah ${checkoutDir}"
        }
    }

    // Build RPM Package if selected
    if (packageType == 'rpm' || packageType == 'all') {
        stage("Build RPM") {
            script {
                sh """
                #!/bin/bash
                set -e  # Stop script on error
                
                APP_NAME="${appName}"
                APP_VERSION="${appVersion}"
                CHECKOUT_DIR="${checkoutDir}"

                echo "Working directory: \$(pwd)"
                echo "Building RPM package for \$APP_NAME version \$APP_VERSION"
                
                # Archive the source directory into a tarball
                tar czvf "\$CHECKOUT_DIR.tar.gz" "\$CHECKOUT_DIR"
                
                # Run rpmbuild with workspace-defined directories
                rpmbuild -ba "\$CHECKOUT_DIR/rpm/\$APP_NAME.spec" --define "_sourcedir \$WORKSPACE"  --define "version \$APP_VERSION"
                
                # Move built RPMs to the workspace for Jenkins archiving
                mkdir -p "\$(pwd)/artifacts"
                find "\${JENKINS_HOME}/rpmbuild/RPMS" "\${JENKINS_HOME}/rpmbuild/SRPMS" -name "*.rpm" -exec mv {} "\$(pwd)/artifacts/" \\
                """
            }
        }
    }

    // Build DEB Package if selected
    if (packageType == 'deb' || packageType == 'all') {
        stage("Build DEB") {
            script {
                sh """
                #!/bin/bash
                set -e  # Stop script on error
                
                APP_NAME="${appName}"
                APP_VERSION="${appVersion}"
                CHECKOUT_DIR="${checkoutDir}"

                echo "Working directory: \$(pwd)"
                echo "Building DEB package for \$APP_NAME version \$APP_VERSION"
                
                # Create a Debian package structure
                mkdir -p "\$CHECKOUT_DIR/deb"
                
                # Assuming a Debian control file is already present in the repo
                cd "\$CHECKOUT_DIR/deb"
                dpkg-buildpackage -us -uc
                
                # Move built DEBs to the workspace for Jenkins archiving
                mkdir -p "\$(pwd)/artifacts"
                mv "../*.deb" "\$(pwd)/artifacts/"
                """
            }
        }
    }

    // Archive Build Artifacts
    stage("Archive Packages") {
        script {
            archiveArtifacts artifacts: '**/*.rpm, **/*.deb', fingerprint: true
        }
    }
}
