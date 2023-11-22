pipeline {
    agent none // Set to none to explicitly control where stages run

    stages {
        stage('Checkout') {
            agent {
                label 'n4c'
            }
            steps {
                checkout scm
            }
        }

        stage('Build and Test on n4c') {
            agent {
                label 'n4c'
            }
            steps {
                sh 'mvn clean package'
                // Additional test steps if needed
            }
        }

        stage('Deploy on n1a') {
            agent any
            steps {
                script {
                    sshagent(credentials: ['n1a']) {
                        deployWithAnsible('n1a', 'ec2-user', '')
                    }
                }
            }
        }

        stage('Deploy on n2u') {
            agent any
            steps {
                script {
                    sshagent(credentials: ['n2u']) {
                        deployWithAnsible('n2u', 'ubuntu', '')
                    }
                }
            }
        }

        stage('Deploy on n3c') {
            agent any
            steps {
                script {
                    sshagent(credentials: ['n3c']) {
                        deployWithAnsible('n3c', 'centos', '')
                    }
                }
            }
        }

        stage('Cleanup') {
            agent any
            steps {
                deleteDir()
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded! Send success notification.'
            script {
                mail to: 'olawalemada@gmail.com',
                     subject: "Success: ${currentBuild.fullDisplayName}",
                     body: "Build, test, and deployment were successful. Congratulations!"
            }
        }
        failure {
            echo 'Pipeline failed! Send failure notification.'
            script {
                mail to: 'olawalemada@gmail.com',
                     subject: "Failed: ${currentBuild.fullDisplayName}",
                     body: "Something went wrong. Please check the build, test, and deployment logs."
            }
        }
    }
}
