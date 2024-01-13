def registry = 'https://nikhilrocky.jfrog.io'
def imageName = 'nikhilrocky.jfrog.io/nikhil-docker-local/ttrend'
def version   = '2.1.2'
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
stage("Jar Publish") {
        steps {
            script {
                    echo '<--------------- Jar Publish Started --------------->'
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"Jfrog-token"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "ttrend-libs-release-local/{1}",
                              "flat": "false",
                              "props" : "${properties}",
                              "exclusions": [ "*.sha1", "*.md5","*.xml"]
                            }
                         ]
                     }"""
                     def buildInfo = server.upload(uploadSpec)
                     buildInfo.env.collect()
                     server.publishBuildInfo(buildInfo)
                     echo '<--------------- Jar Publish Ended --------------->'  
            
            }
        }   
    }
    stage(" Docker Build ") {
      steps {
        script {
           echo '<--------------- Docker Build Started --------------->'
           app = docker.build(imageName+":"+version)
           echo '<--------------- Docker Build Ends --------------->'
        }
      }
    }
    stage (" Docker Publish "){
        steps {
            script {
               echo '<--------------- Docker Publish Started --------------->'  
                docker.withRegistry(registry, 'Jfrog-token'){
                    app.push()
                }    
               echo '<--------------- Docker Publish Ended --------------->'  
            }
        }

    }
}
}
