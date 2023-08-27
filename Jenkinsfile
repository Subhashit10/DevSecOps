pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git changelog: false, poll: false, url: 'https://github.com/shubnimkar/CI_CD_Devsecops.git'
            }
        }

        stage ('Check secrets') {
          steps {
              sh 'docker run --rm -it -v " https://github.com/shubnimkar/CI_CD_Devsecops.git" trufflesecurity/trufflehog:latest github --org=trufflesecurity -format json -output truffelhog_output.json || true'
              sh 'trufflehog3 -R report.json --output report.html'
      }
         
    }
}
}

