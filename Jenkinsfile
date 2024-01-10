pipeline {
    agent {
        node {
            label 'Maven-Agent'
        }
    }
    environment {
        PATH = "/opt/apache-maven-3.9.6/bin:$PATH"
    }

    stages {
        stage('Build the code') {
            steps {
                sh 'mvn clean install'
            }
        }
    
    stage('Sonarqube Analysis') {
    environment {
        scannerHome = tool 'SonarQube-Scanner'
    }
    steps {
        withSonarQubeEnv('SonarQube-Server') {
            sh "${scannerHome}/bin/sonar-scanner"
        }
    }
}
}
}
