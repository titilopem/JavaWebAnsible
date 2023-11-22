pipeline {
    agent none

    environment {
        // Define credentials IDs for nodes
        N1A_CREDENTIAL = credentials('n1a')
        N2U_CREDENTIAL = credentials('n2u')
        N3C_CREDENTIAL = credentials('n3c')
    }

    stages {
        stage('Checkout') {
            agent any
            steps {
                node {
                    checkout scm
                }
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
                    deployWithAnsible('n1a', 'ec2-user', N1A_CREDENTIAL)
                }
            }
        }

        stage('Deploy on n2u') {
            agent any
            steps {
                script {
                    deployWithAnsible('n2u', 'ubuntu', N2U_CREDENTIAL)
                }
            }
        }

        stage('Deploy on n3c') {
            agent any
            steps {
                script {
                    deployWithAnsible('n3c', 'centos', N3C_CREDENTIAL)
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

def deployWithAnsible(host, user, credentialId) {
    node {
        sshagent([credentialId]) {
            ansiblePlaybook(
                become: true,
                inventory: 'hosts.ini',
                playbook: 'javawebansible.yml',
                extraVars: [
                    war_file: 'target/*.war',
                    ansible_user: user,
                    ansible_ssh_common_args: '-o StrictHostKeyChecking=no',
                    ANSIBLE_DEBUG: '-vvv' // Add this line for increased verbosity
                ]
            )
        }
    }
}
