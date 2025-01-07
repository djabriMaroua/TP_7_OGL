pipeline {
    agent any

    environment {
        SONAR_HOST_URL = 'http://localhost:9000/'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/djabriMaroua/TP_7_OGL.git'
            }
        }

      stage('Test') {
                  steps {
                      echo 'Running unit tests and generating reports...'
                      script {
                          try {
                              // Étape 1 : Exécuter les tests unitaires
                              bat './gradlew test'

                              // Étape 2 : Archiver les résultats des tests unitaires


                              // Étape 3 : Générer les rapports Cucumber


                              // Étape 4 : Archiver les rapports Cucumber générés
                              archiveArtifacts artifacts: 'build/reports/cucumber/**/*', fingerprint: true
                          } catch (Exception e) {
                              echo "Test stage failed: ${e.message}"
                              currentBuild.result = 'FAILURE'
                              error("Test stage failed")
                          }
                      }
                  }
              }

        stage('Code Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                script {
                    withSonarQubeEnv('sonar') {
                        bat './gradlew sonar'
                    }
                }
            }
        }

        stage('Code Quality') {
            steps {
                echo 'Checking SonarQube Quality Gates...'
                script {
                    try {
                        timeout(time: 3, unit: 'MINUTES') {
                            def qg = waitForQualityGate()


                            if (qg.status != 'OK') {
                                echo "Quality Gates failed: ${qg.status}"
                                currentBuild.result = 'UNSTABLE'
                                error("Quality Gates failed. Stopping pipeline.")
                            } else {
                                echo "Quality Gates passed: ${qg.status}"
                            }
                        }
                    } catch (Exception e) {
                        echo "Quality Gates check failed: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        error("Quality Gates check failed")
                    }
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Building the project...'
                script {
                    try {
                        bat './gradlew build'
                        archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true
                    } catch (Exception e) {
                        echo "Build stage failed: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        error("Build stage failed")
                    }
                }
            }
        }


        stage('Deployy') {
            steps {
                echo 'Deploying to MyMavenRepo...'
                bat "./gradlew publish"
            }
        }

      stage('Notifications') {
                  steps {
                      script {
                          def result = currentBuild.result ?: 'SUCCESS'

                          // Envoyer une notification par e-mail
                          if (result == 'SUCCESS') {
                              mail to: 'lm_djabri@esi.dz',
                                   subject: "Jenkins Build #${env.BUILD_NUMBER} Success",
                                   body: "The build #${env.BUILD_NUMBER} was successful.\n\nCheck it out: ${env.BUILD_URL}"
                          } else {
                              mail to: 'lm_djabri@esi.dz',
                                   subject: "Jenkins Build #${env.BUILD_NUMBER} Failure",
                                   body: "The build #${env.BUILD_NUMBER} failed.\n\nCheck it out: ${env.BUILD_URL}"
                          }

                          // Envoyer une notification Slack
                          if (result == 'SUCCESS') {
                              slackSend channel: '#tpogl',
                                        color: 'good',
                                        message: "Build ${env.JOB_NAME} #${env.BUILD_NUMBER} completed successfully."
                          } else {
                              slackSend channel: '#tpogl',
                                        color: 'danger',
                                        message: "Build ${env.JOB_NAME} #${env.BUILD_NUMBER} failed. Check the logs: ${env.BUILD_URL}"
                          }
                      }
                  }

          }
    post {
        always {
            echo 'Pipeline execution finished.'
        }

        success {
            echo 'Pipeline succeeded!'
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}