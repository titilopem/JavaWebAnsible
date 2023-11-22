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
            agent any
            steps {
                script {
                    ansiblePlaybook(
                        become: true,
                        inventory: 'hosts.ini',
                        playbook: 'javawebansible.yml',
                        extraVars: [
                            war_file: 'target/*.war',
                            ansible_user: 'ec2-user',
                            ansible_ssh_private_key_file: '/home/centos/doorkey.pem',
                            ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
                        ]
                    )
                }
            }
        }

        stage('Deploy on n2u') {
            agent any
            steps {
                script {
                    ansiblePlaybook(
                        become: true,
                        inventory: 'hosts.ini',
                        playbook: 'javawebansible.yml',
                        extraVars: [
                            war_file: 'target/*.war',
                            ansible_user: 'ubuntu',
                            ansible_ssh_private_key_file: '/home/centos/doorkey.pem',
                            ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
                        ]
                    )
                }
            }
        }

        stage('Deploy on n3c') {
            agent any
            steps {
                script {
                    ansiblePlaybook(
                        become: true,
                        inventory: 'hosts.ini',
                        playbook: 'javawebansible.yml',
                        extraVars: [
                            war_file: 'target/*.war',
                            ansible_user: 'centos',
                            ansible_ssh_private_key_file: '/home/centos/doorkey.pem',
                            ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
                        ]
                    )
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
