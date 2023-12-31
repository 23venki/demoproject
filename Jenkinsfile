def registry = 'https://venki01.jfrog.io'
def imageName = 'venki01.jfrog.io/venki-docker-local/demo'
def version   = '2.1.2'
pipeline {
       agent {
           node {
               label 'maven-agent'
           }
       }
environment {
    PATH = "/opt/apache-maven-3.9.3/bin:$PATH"
       
}
    stages {
        stage('build') {
            steps {
                sh 'mvn clean deploy -Dmaven.test.skip=true'
            }
        }

        stage('unit test') {
            steps{
                echo '-------Unit test started--------'
                sh 'mvn surefire-report:report'
                echo '-------Unit test completed--------'
            }
        }
        stage('SonarQube analysis') {
            environment{
            scannerHome = tool 'SonarScanner'
       }
        steps {
        withSonarQubeEnv('SonarQube-server') { // If you have configured more than one global server connection, you can specify its name
            sh "${scannerHome}/bin/sonar-scanner"
       }
    }
  }
        stage("Quality Gate"){
            steps {
                script {
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
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"Artifactory-token"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "demo-libs-release-local/{1}",
                              "flat": "false",
                              "props" : "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
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
                 docker.withRegistry(registry, 'Artifactory-token'){
                    app.push()
                 }    
                echo '<--------------- Docker Publish Ended --------------->'  
            }
        }
    }
 }
}
