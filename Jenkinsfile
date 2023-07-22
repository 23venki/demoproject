pipeline {
       agent {
           node {
               label 'maven-agent'
           }
       }

    stages {
        stage('Hello') {
            steps {
                git branch: 'main', url: 'https://github.com/23venki/demoproject.git'
            }
        }
    }
}
