pipeline {
    agent {
        label "default-1"  // The label corresponds to your worker node
    }    
   tools{
         go '1.24.0'
         }
    stages {
       
       

      stage('Build') {
            steps {
                sh 'go version'
            }
        }

       stage('Last stage') {
            steps {
                // Example build command, adjust as needed
                echo 'END'
            }
        }
    }
}

