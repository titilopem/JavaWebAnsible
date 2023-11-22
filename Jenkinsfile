pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Test on n4c') {
            agent {
                label 'n4c'
            }
            steps {
                script {
                    sh 'mvn clean package'
                    // Additional test steps if needed
                }
            }
        }

        stage('Deploy on Nodes') {
            parallel {
                stage('Deploy on n1a') {
                    agent {
                        label 'n1a'
                    }
                    steps {
                        deployWithAnsible('n1a', 'ec2-user', credentials('n1a'))
                    }
                }
                stage('Deploy on n2u') {
                    agent {
                        label 'n2u'
                    }
                    steps {
                        deployWithAnsible('n2u', 'ubuntu', credentials('n2u'))
                    }
                }
                stage('Deploy on n3c') {
                    agent {
                        label 'n3c'
                    }
                    steps {
                        deployWithAnsible('n3c', 'centos', credentials('n3c'))
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                cleanWs()
            }
        }
        success {
            echo 'Pipeline succeeded! Send success notification.'
            emailext subject: "Success: ${currentBuild.fullDisplayName}",
                     body: "Build, test, and deployment were successful. Congratulations!",
                     to: 'olawalemada@gmail.com'
        }
        failure {
            echo 'Pipeline failed! Send failure notification.'
            emailext subject: "Failed: ${currentBuild.fullDisplayName}",
                     body: "Something went wrong. Please check the build, test, and deployment logs.",
                     to: 'olawalemada@gmail.com'
        }
    }
}
