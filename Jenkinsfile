pipeline {
    agent {
        label "default-1"  // The label corresponds to your worker node
    }    
   tools{
         go '1.24.0'
         }
    stages {
       
        stage('Last stage') {
            steps {
                // Example build command, adjust as needed
                echo 'ok'
            }
        }

      stage('Build') {
            steps {
                sh 'go version'
            }
        }
    }
}

