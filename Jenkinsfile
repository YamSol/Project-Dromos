pipeline {
    agent any
    
    environment {
        DEFAULT_RECIPIENTS = 'yambertamini.sol@gmail.com'
    }
    
    stages {
        
        stage('Check Environment') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'java -version'
                        sh 'mvn -version'
                    } else {
                        bat 'java -version'
                        bat 'mvn -version'
                    }
                }
            }
        }
        
        stage('Test') {
            steps {
                dir('backend') {
                    script {
                        if (isUnix()) {
                            sh 'mvn test'
                        } else {
                            bat 'mvn test'
                        }
                    }
                }
            }
            post {
                always {
                    junit testResults: 'backend/target/surefire-reports/*.xml',
                          allowEmptyResults: true
                }
            }
        }
        
        stage('Build') {
            steps {
                dir('backend') {
                    script {
                        if (isUnix()) {
                            // Linux/Unix/Mac
                            sh 'mvn clean package'
                        } else {
                            // Windows
                            bat 'mvn clean package'
                        }
                    }
                }
            }
            post {
                always {
                    // Arquiva os artifacts da build
                    archiveArtifacts artifacts: 'backend/target/*.jar', 
                                   fingerprint: true,
                                   allowEmptyArchive: true
                    
                    // Publica resultados dos testes
                    junit testResults: 'backend/target/surefire-reports/*.xml',
                          allowEmptyResults: true
                    
                    // Arquiva logs da build
                    archiveArtifacts artifacts: 'backend/target/maven-status/**/*', 
                                   fingerprint: false,
                                   allowEmptyArchive: true
                }
            }
        }
    }

    post {
        always {
            script {
                // Envia email sempre (com resultado da build)
                try {
                    def buildStatus = currentBuild.currentResult ?: 'SUCCESS'
                    def subject = "Jenkins Build ${buildStatus}: ${env.JOB_NAME} - #${env.BUILD_NUMBER}"
                    def body = """
Build ${buildStatus}!

Job: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Build URL: ${env.BUILD_URL}
Duration: ${currentBuild.durationString}

${buildStatus == 'SUCCESS' ? 'Build completed successfully!' : 'Build failed!'}

Artifacts: ${env.BUILD_URL}artifact/
Console Output: ${env.BUILD_URL}console/
                    """.trim()

                    // Tenta enviar email (só funciona se SMTP estiver configurado)
                    emailext (
                        subject: subject,
                        body: body,
                        to: env.DEFAULT_RECIPIENTS,
                        mimeType: 'text/plain',
                        attachLog: true,
                        compressLog: true
                    )
                    echo "Email enviado com sucesso!"
                } catch (Exception e) {
                    echo "Não foi possível enviar email: ${e.message}"
                    echo "Configure SMTP em Manage Jenkins > Configure System > E-mail Notification"
                }
            }
            
            // Limpa o workspace (mantém artifacts arquivados)
            cleanWs()
        }
        success {
            echo 'Build completed successfully!'
            
            // Email específico de sucesso (opcional)
            script {
                try {
                    mail (
                        to: env.DEFAULT_RECIPIENTS,
                        subject: "SUCCESS: ${env.JOB_NAME} - #${env.BUILD_NUMBER}",
                        body: "Build completed successfully!\n\nCheck artifacts: ${env.BUILD_URL}artifact/"
                    )
                } catch (Exception e) {
                    echo "Email simples falhou, mas emailext já tentou enviar."
                }
            }
        }
        failure {
            echo 'Build failed!'
            
            // Email específico de falha (opcional)
            script {
                try {
                    mail (
                        to: env.DEFAULT_RECIPIENTS,
                        subject: "FAILURE: ${env.JOB_NAME} - #${env.BUILD_NUMBER}",
                        body: "Build failed!\n\nCheck console: ${env.BUILD_URL}console/"
                    )
                } catch (Exception e) {
                    echo "Email simples falhou, mas emailext já tentou enviar."
                }
            }
        }
        unstable {
            echo 'Build unstable!'
        }
    }
}