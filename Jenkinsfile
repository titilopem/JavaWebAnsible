// Function to deploy with Ansible
def deployWithAnsible(host, user, privateKey) {
    ansiblePlaybook(
        become: true,
        inventory: 'hosts.ini',
        playbook: 'javawebansible.yml',
        extraVars: [
            war_file: 'target/*.war',
            ansible_user: user,
            ansible_ssh_private_key_file: privateKey,
            ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
        ]
    )
}

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
                sh 'mvn clean package'
                // Additional test steps if needed
            }
        }

        stage('Deploy on n1a') {
            steps {
                script {
                    deployWithAnsible('n1a', 'ec2-user', '/home/centos/doorkey.pem')
                }
            }
        }

        stage('Deploy on n2u') {
            steps {
                script {
                    deployWithAnsible('n2u', 'ubuntu', '/home/centos/doorkey.pem')
                }
            }
        }

        stage('Deploy on n3c') {
            steps {
                script {
                    deployWithAnsible('n3c', 'centos', '/home/centos/doorkey.pem')
                }
            }
        }

        stage('Cleanup') {
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
