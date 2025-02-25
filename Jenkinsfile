pipeline {
    agent {
        label "default-1"  // The label corresponds to your worker node
    }    
    tools {
        go '1.24.0'  // Specify the Go version to use in the pipeline
   nodejs "node-23.8.0-from.org"
    }
    stages {
        stage('Build') {
            steps {
                sh 'go version'  // This step ensures that Go is installed and the version is displayed
            }
        }

        stage('Run Fuzz Test') {
            steps {
                // Run the fuzz test
                sh 'go test -fuzz=FuzzMarkdownRenderRaw -fuzztime=10s ./tests/fuzz/'
            }
        }

        stage('Last stage') {
            steps {
                // Example build command, adjust as needed
                sh 'npx vitest run'
                echo 'END'
            }
        }
    }
}
