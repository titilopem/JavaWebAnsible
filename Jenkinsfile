pipeline {
    agent any

    environment {
        N1A_CREDENTIAL = credentials('n1a')
        N2U_CREDENTIAL = credentials('n2u')
        N3C_CREDENTIAL = credentials('n3c')
    }

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
                        deployWithAnsible('n1a', 'ec2-user', N1A_CREDENTIAL)
                    }
                }
                stage('Deploy on n2u') {
                    agent {
                        label 'n2u'
                    }
                    steps {
                        deployWithAnsible('n2u', 'ubuntu', N2U_CREDENTIAL)
                    }
                }
                stage('Deploy on n3c') {
                    agent {
                        label 'n3c'
                    }
                    steps {
                        deployWithAnsible('n3c', 'centos', N3C_CREDENTIAL)
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
