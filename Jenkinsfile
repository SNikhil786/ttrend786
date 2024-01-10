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
                echo '------------Build Started-----------------'
                sh 'mvn install -Dmaven.test.skip=true'
                echo '------------Build Completed-----------------'
            }
        }

        stage('Unit Test') {
            steps {
                echo '------------UnitTEst Started-----------------'
                sh 'mvn surefire-report:report'
                echo '------------UnitTEst Completed-----------------'
            }
        }

    
    stage('Sonarqube Analysis') {
    environment {
        scannerHome = tool 'SonarQube-Scanner'
    }
    steps {
        echo '------------Sonar Started-----------------'
        withSonarQubeEnv('SonarQube-Server') {
            sh "${scannerHome}/bin/sonar-scanner"
        }
        echo '------------Sonar Completed-----------------'
    }
}
    stage('QualityGate Checking')
    {
        steps{
            script{
                timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
                    def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
                    if (qg.status != 'OK') {
                    error "Pipeline aborted due to quality gate failure: ${qg.status}"
            }
        }
    }
}
}
    }
}
