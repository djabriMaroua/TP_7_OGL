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
                echo 'Running unit tests...'
                script {
                    try {
                        bat './gradlew test'
                        junit '**/build/test-results/test/*.xml'
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
                                   // Step 1: Get the task ID from the report-task.txt file
                                   def reportTaskFile = readFile('.scannerwork/report-task.txt')
                                   def taskId = reportTaskFile.find(/ceTaskId=(.*)/) { it[1] }

                                   // Step 2: Call the SonarQube API to check the task status
                                   def taskStatus = ''
                                   while (true) {
                                       def taskResponse = sh(script: "curl -u ${SONAR_AUTH_TOKEN}: '${SONAR_HOST_URL}/api/ce/task?id=${taskId}'", returnStdout: true)
                                       taskStatus = new groovy.json.JsonSlurper().parseText(taskResponse).task.status

                                       if (taskStatus == 'SUCCESS' || taskStatus == 'FAILED' || taskStatus == 'CANCELED') {
                                           break
                                       }
                                       sleep(time: 5, unit: 'SECONDS') // Wait before checking again
                                   }

                                   if (taskStatus != 'SUCCESS') {
                                       error("SonarQube analysis task failed with status: ${taskStatus}")
                                   }

                                   // Step 3: Call the SonarQube API to check the Quality Gate status
                                   def qualityGateResponse = sh(script: "curl -u ${SONAR_AUTH_TOKEN}: '${SONAR_HOST_URL}/api/qualitygates/project_status?projectKey=your-project-key'", returnStdout: true)
                                   def qualityGateStatus = new groovy.json.JsonSlurper().parseText(qualityGateResponse).projectStatus.status

                                   // Step 4: Handle the Quality Gate status
                                   if (qualityGateStatus == 'OK') {
                                       echo "Quality Gates passed: ${qualityGateStatus}"
                                   } else if (qualityGateStatus == 'WARN') {
                                       echo "Quality Gates warning: ${qualityGateStatus}"
                                       currentBuild.result = 'UNSTABLE'
                                   } else if (qualityGateStatus == 'ERROR') {
                                       echo "Quality Gates failed: ${qualityGateStatus}"
                                       currentBuild.result = 'FAILURE'
                                       error("Quality Gates failed. Stopping pipeline.")
                                   } else {
                                       echo "Unknown Quality Gate status: ${qualityGateStatus}"
                                       currentBuild.result = 'FAILURE'
                                       error("Unknown Quality Gate status. Stopping pipeline.")
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

        stage('Send Notification') {
            steps {
                script {
                    def result = currentBuild.result
                    if (result == 'SUCCESS') {
                        mail to: 'lm_djabri@esi.dz',
                             subject: "Jenkins Build #${env.BUILD_NUMBER} Success",
                             body: "The build #${env.BUILD_NUMBER} was successful.\n\nCheck it out: ${env.BUILD_URL}"
                    } else {
                        mail to: 'lm_djabri@esi.dz',
                             subject: "Jenkins Build #${env.BUILD_NUMBER} Failure",
                             body: "The build #${env.BUILD_NUMBER} failed.\n\nCheck it out: ${env.BUILD_URL}"
                    }
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