pipeline {
    agent {
        label "w-1"
    }

    environment {
        GIT_TAG = sh(script: "git describe --tags --abbrev=0", returnStdout: true).trim()
    }

    tools {
        nodejs "node-23.8.0-from.org"
        go '1.24.0'
    }

    stages {
        // Stage for preparing environment and running tests
        stage('Run Tests') {
            steps {
                script {
                    // Ensure dependencies are up to date before testing and linting
                    sh 'go mod tidy'

                    parallel(
                        'Run Fuzz Test': {
                            // Run the fuzz test
                            sh 'go test -fuzz=FuzzMarkdownRenderRaw -fuzztime=10s ./tests/fuzz/'
                        },
                        'Lint Go Code': {
                            // Run golangci-lint on all Go files
                            sh 'golangci-lint run'
                        }
                    )
                }
            }
        }

        // Stage for running Vitest tests
        stage('Vitest') {
            steps {
                // Run vitest tests
                sh 'npx vitest run'
                echo 'END'
            }
        }

        // Parallel stage for building Go and NPM
        stage('Parallel Build') {
            parallel {
                stage('NPM Install') {
                    steps {
                        script {
                            sh 'npm install'
                        }
                    }
                }

                stage('Go Build') {
                    steps {
                        script {
                            // Go build after ensuring dependencies are tidy
                            sh 'go build -ldflags="-s -w" -o gitea'
                        }
                    }
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

        // Archive the files into a tar.gz archive
        stage('Create TAR Archive') {
            steps {
                script {
                    def filesToInclude = [
                        "web_src/", "templates/", "models/", "public/", "options/", 
                        "node_modules/", "modules/", "gitea", "custom/"
                    ]
                    def tarCommand = "tar -czf gitea-archive-${GIT_TAG}.tar.gz -C $WORKSPACE "
                    filesToInclude.each { file -> 
                        tarCommand += "${file} "
                    }
                    sh tarCommand
                }
            }
        }

        // Upload the archive to S3
        stage('Upload to S3') {
            steps {
                script {
                    s3Upload(
                        bucket: 'my-artifact-bucket-ue-north-1/${JOB_NAME}-${BUILD_NUMBER}',
                        sourceFile: "gitea-archive-${GIT_TAG}.tar.gz",
                        storageClass: 'STANDARD',
                        selectedRegion: 'eu-north-1',
                        gzipFiles: false,
                        flatten: false,
                        showDirectlyInBrowser: false,
                        noUploadOnFailure: true,
                        useServerSideEncryption: false,
                        keepForever: false,
                        managedArtifacts: false,
                        entries: [[
                            sourceFile: "gitea-archive-${GIT_TAG}.tar.gz",
                            bucket: "my-artifact-bucket-ue-north-1/${JOB_NAME}-${BUILD_NUMBER}",
                            excludedFile: "",
                            flatten: false,
                            gzipFiles: false,
                            keepForever: false,
                            managedArtifacts: false,
                            noUploadOnFailure: true,
                            selectedRegion: 'eu-north-1',
                            showDirectlyInBrowser: false,
                            storageClass: 'STANDARD',
                            useServerSideEncryption: false
                        ]],
                        profileName: 's3-jenkins-ansible-user', 
                        pluginFailureResultConstraint: 'FAILURE'
                    )
                }
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
