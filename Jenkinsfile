pipeline {
    agent any
    tools {
        maven "maven3"
        jdk "java8"
    } 

    environment {
        SNAP_REPO = 'v_snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'admin123'
        RELEASE_REPO = 'v_artifact'
        CENTRAL_REPO = 'v_maven_dependency'
        NEXUSIP = '172.31.0.180'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'v_group'
        NEXUS_LOGIN = 'nexuslogin'
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
    }

    stages {
        stage('build'){
            steps {
                sh 'mvn -s settings.xml -Dskiptests install'
            }

            post {
                success {
                    echo "now archiving"
                    archiveArtifacts artifacts: '**/*.war'
                }  
            }
        }

        stage('test'){
            steps {
                sh 'mvn -s settings.xml install'
            }
        }
        
        stage('checkstyle analysis'){
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

         stage('CODE ANALYSIS with SONARQUBE') {
          
		  environment {
             scannerHome = tool "${SONARSCANNER}"
          }

          steps {
            withSonarQubeEnv("${SONARSERVER}") {
               sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
           }  
       }

       stage("Quality Gates") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate : true
                }
            }
       }
    }
}
