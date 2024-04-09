  COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline {
    agent any
    tools {
        maven "maven3.9.6"
    }
    
    stages {
        stage ("Git clone") {
            steps {
                git branch: 'main', url: 'https://github.com/micaidoo/web-app.git'
            }
        }

        stage ("build with Maven") {
            steps {
                sh "mvn clean"
            }
        }

        stage ("Testing with Maven") {
            steps {
                sh "mvn test"
            }
        }

        stage ("Package with Maven") {
            steps {
                sh "mvn package"
            }
        }

        stage('SonarQube Analysis') {
            environment {
                ScannerHome = tool 'sonar5.0'
            }
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        sh "${ScannerHome}/bin/sonar-scanner -Dsonar.projectKey=jomacs"
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
          }
      }

        stage ("Upload to Nexus") {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'maven-web-application', classifier: '', file: '/var/lib/jenkins/workspace/first pipeline job/target/web-app.war', type: 'war']], credentialsId: 'nexus-id', groupId: 'com.mt', nexusUrl: '18.221.65.60:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'webapp-release', version: '3.1.2-RELEASE'
            }
        }

        stage ("Deploy to UAT") {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'tomcat-credential', path: '', url: 'http://13.59.27.67:8080/')], contextPath: null, war: 'target/*.war'
            }
        }
    }

    post {
        always {
            slackSend channel: 'team-usa', 
                      color: COLOR_MAP[currentBuild.currentResult],
                      message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}"
        }
    }
}