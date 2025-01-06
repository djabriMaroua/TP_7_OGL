pipeline {
    agent any

    stages {
        stage('Run Tests') {
            steps {
                bat 'gradlew test'
            }
        }
        stage('Generate Cucumber Reports') {
            steps {
                bat 'gradlew generateCucumberReports'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    bat 'gradlew sonarqube'
                }
            }
        }
        stage('Code Quality') {
            steps {
                script {
                    def qualityGate = waitForQualityGate()
                    if (qualityGate.status != 'OK') {
                        error "Pipeline failed due to Quality Gate failure: ${qualityGate.status}"
                    }
                }
            }
        }
        stage('Build Jar') {
            steps {
                bat 'gradlew build'
            }
        }
        stage('Generate Documentation') {
            steps {
                bat 'gradlew generateJavadoc'
            }
        }
        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true
                archiveArtifacts artifacts: 'build/docs/javadoc/**/*', fingerprint: true
            }
        }
        stage('Deploy') {
            steps {
                script {
                    bat 'gradlew.bat publish'
                }
            }
        }
    }

    post {
             success {


                     mail(
                         to: 'lm_djabri@esi.dz',
                         subject: 'Deployment Success - Project Nina ',
                         body: 'The deployment for the project Nina was successful.'
                            )
                       slackSend(
                                            channel: '#dev',
                                             color: 'good',
                                             message: 'Deployment succeeded for project Thanina-ci-cd!'
                                            )

                    }
                   failure {
                                        // Email Notification for Pipeline Failure
                  mail(
                    to: 'lm_djabri@esi.dz',
                     subject: 'Pipeline Failed - Project Nina',
                     body: 'The Jenkins pipeline for project Nina has failed. Please check the logs for more details.'
                    )
                  slackSend(
                                                              channel: '#dev',
                                                               color: 'good',
                                                               message: 'Deployment failed for project Thanina-ci-cd!'
                                                              )


                  }
             }
}


