pipeline {
    agent {
        label "default-1"
    }

    environment {
        // Get the latest Git tag for versioning
        GIT_TAG = sh(script: "git describe --tags --abbrev=0", returnStdout: true).trim()
    }

    stages {

        // Install npm dependencies
        stage('NPM Install') {
            steps {
                script {
                    sh 'npm install'
                }
            }
        }

        // Run webpack with the specified config
        stage('Webpack Build') {
            steps {
                script {
                    sh 'npx webpack --config webpack.config.js'
                }
            }
        }

        // Run Go mod tidy and build the Go binary with minimized size
        stage('Go Build') {
            steps {
                script {
                    // Run Go mod tidy to clean up any dependencies
                    sh 'go mod tidy'

                    // Build the Go binary, using flags to reduce binary size
                    // -ldflags="-s -w" removes symbol and DWARF debugging information, making the binary smaller
                    sh 'go build -ldflags="-s -w" -o gitea'
                }
            }
        }

        // Create a tar.gz archive of your project
        stage('Create TAR Archive') {
            steps {
                script {
                    // Define the files to include relative to the workspace
                    def filesToInclude = [
                        "web_src/",   
                        "templates/", 
                        "models/", 
                        "public/", 
                        "options/", 
                        "node_modules/", 
                        "modules/", 
                        "gitea",  // The built Go binary
                        "custom/"     
                    ]

                    // Build the tar command dynamically, using -C to avoid absolute paths
                    def tarCommand = "tar -czf gitea-archive-${GIT_TAG}.tar.gz -C $WORKSPACE "
                    filesToInclude.each { file ->
                        tarCommand += "${file} "
                    }

                    // Run the tar command
                    sh tarCommand
                }
            }
        }

        // Publish the versioned artifact to S3
        stage('Publish to Remote Server') {
            steps {
                // Upload the versioned artifact to S3
                s3Upload consoleLogLevel: 'INFO', dontSetBuildResultOnFailure: false, dontWaitForConcurrentBuildCompletion: false, entries: [[
                    bucket: "my-artifact-bucket-ue-north-1/${JOB_NAME}-${BUILD_NUMBER}",
                    excludedFile: '', 
                    flatten: false, 
                    gzipFiles: false, 
                    keepForever: false, 
                    managedArtifacts: false, 
                    noUploadOnFailure: true, 
                    selectedRegion: 'us-east-1', 
                    showDirectlyInBrowser: false, 
                    sourceFile: "${env.WORKSPACE}/gitea-archive-${GIT_TAG}.tar.gz",  // Use versioned file
                    storageClass: 'STANDARD', 
                    uploadFromSlave: false, 
                    useServerSideEncryption: false,
                    pluginFailureResultConstraint: 'FAILURE', 
                    profileName: 's3-jenkins-ansible-user', 
                    userMetadata: [[key: '', value: '']]
                ]]

                // Add the pluginFailureResultConstraint, profileName, and userMetadata as separate blocks
               
            }
        }
    }

    post {
        success {
            echo "Release ${GIT_TAG} has been successfully uploaded to S3."
        }
        failure {
            echo "Pipeline failed. Check the logs for more details."
        }
    }
}

