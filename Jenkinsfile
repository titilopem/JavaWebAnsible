pipeline {
    agent any

    environment {
        N4C_CREDENTIAL = credentials('n3c')
        N6C_CREDENTIAL = credentials('n3c')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            agent {
                label 'n4c'  // Use the n4c node for build
            }
            steps {
                echo 'Building the project using N4C'
                script {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Test') {
            agent {
                label 'n4c'  // Use the n4c node for testing
            }
            steps {
                echo 'Running tests using N4C'
                script {
                    sh 'mvn test'
                    stash (name: 'build', includes: "target/*.war")
                }
            }
        }

        stage('Deploy on Ansible Master') {
            agent {
                label 'n6c'
            }
            steps {
                script {
                    echo 'Deploying on Ansible Master'
                    unstash 'build'
                    sh 'ansible-playbook -i hosts.ini -e "war_file=target/*.war" javawebansible.yml'
                }
            }
        }
    }

    post {
        always {
            script {
                //cleanWs()
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
