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
              sh 'docker run gesellix/trufflehog --json https://github.com/shubnimkar/CI_CD_Devsecops.git > trufflehog'
              sh 'cat trufflehog'
      }
         
    }
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Code Compile') {
            steps {
                sh "mvn clean compile"
            }
        }
        
        stage('Unit Tests') {
            steps {
                sh "mvn test"
            }
        }
        
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petclinic \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Petclinic '''
    
                }
            }
        }

        stage("Build"){
            steps{
                sh " mvn clean install"
            }
        }
        
        stage("Docker Build"){
            steps{
                script{
                   withDockerRegistry([url:'https://index.docker.io/v1/',credentialsId: '69fb7f6f-90ba-4bab-baf3-765387680986', toolName: 'docker']) {
                        sh "docker build -t petclinic1 ."
                    }
                }
            }
        }
        
        stage("Docker Tag & Push"){
            steps{
                script{
                   withDockerRegistry([url:'https://index.docker.io/v1/',credentialsId: '69fb7f6f-90ba-4bab-baf3-765387680986', toolName: 'docker']) {
                        sh "docker tag petclinic1 shubnimkar/pet-clinic25:latest"
                        sh "docker push shubnimkar/pet-clinic25:latest"
                    }
                }
            }
        }
        
        stage("Deploy To Tomcat"){
            steps{
            
                sh "cp  /var/lib/jenkins/workspace/DevSecOps/target/petclinic.war /opt/apache-tomcat-9.0.65/webapps/ "
            }
        
        }
        stage('Run ZAP Scan') {
            steps {
                // Start OWASP ZAP and run a security scan
                sh '''
                    # Start ZAP in headless mode
                    docker run -d --name zap-container -v $(pwd):/zap/wrk/:rw -t owasp/zap2docker-stable zap.sh -cmd -daemon -host 0.0.0.0 -port 8090 -config api.disablekey=true

                    # Run ZAP Baseline Scan
                    docker exec zap-container zap-baseline.py -t http://3.108.238.36:8081/petclinic -r zap-report_file

                    # Stop the ZAP container
                    docker stop zap-container
                '''
            }
        }
}
}

