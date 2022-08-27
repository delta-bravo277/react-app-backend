pipeline {
    
    agent any 
    
    environment {
  
        DOCKERHUB_REPO='sanketlawande1/node-app'
	    DOCKERHUB_CREDENTIALS=credentials('DOCKERHUB_CREDENTIALS')
	    SECRET=credentials('SECRET')
    }


stages {
    
    stage("Git Checkout") {

        steps { git branch: 'main', url: 'https://github.com/delta-bravo277/node-app' }
        
    }
    
    stage('Code Scanning') {
        
        steps { echo 'sonar-scanner' }
    }
    
    stage('Installing Dependencies') {
        
         steps { sh 'npm install' }
    }
    
    stage('Dependencies Scanning and Fixing') {
        
       steps { 
        sh 'npm audit fix || true '
        sh 'npm audit --audit-level=none || true'
        sh 'npm audit --json | npm-audit-html'
        
        publishHTML target: [
            allowMissing: false,
            alwaysLinkToLastBuild: false,
            keepAll: true,
            reportDir: '.',
            reportFiles: 'npm-audit.html',
            reportName: 'NPM Audit Report'
            ]
       } // steps scanning closed
    } // stage scanning closed
    
    stage('Build , Sign and Publish an image') {
        
        steps {
            
            sh '''
                docker build -t $DOCKERHUB_REPO:$BUILD_NUMBER .
                echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                export DOCKER_CONTENT_TRUST_REPOSITORY_PASSPHRASE=$SECRET
                docker trust sign $DOCKERHUB_REPO:$BUILD_NUMBER
                
                '''
                        
        } // steps build and sign closed
         
    
    } //  stage build and sign closed

    stage('Scanning an Image') {
        steps {

            sh 'clair-scanner -r clair-report.html --ip 127.17.0.1 $DOCKERHUB_REPO:$BUILD_NUMBER'
        } // steps scanning-image closed
    } // stage scanning-image closed 
    
} // stages closed

} // pipeline closed
