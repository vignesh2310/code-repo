def COLOR_MAP = [
      'SUCCESS': 'good',
      'FAILURE': 'danger',
]

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

       stage("UploadArtifact") {
           steps {
                nexusArtifactUploader(
                 nexusVersion: 'nexus3',
                 protocol: 'http',
                 nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                 groupId: 'QA',
                 version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                 repository: "${RELEASE_REPO}",
                 credentialsId: "${NEXUS_LOGIN}",
           artifacts: [
                 [artifactId: 'v_pro_artifact',
                  classifier: '',
                  file: 'target/vprofile-v2.war',
                  type: 'war']
                      ]
                )
            }
        }
    }
    post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#devops',
               color: COLOR_MAP[currentBuild.currentResult],
               message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More Info At: ${env.BUILD_URL}"
        }
    }

}
