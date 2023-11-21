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
                stash(name: 'war', includes: 'target/*.war')
                // Additional test steps if needed
            }
        }

        stage('Deploy on n1a') {
            steps {
                script {
                    unstash 'war'

                    ansiblePlaybook(
                        become: true,
                        inventory: 'hosts.ini',
                        playbook: 'javawebansible.yml',
                        extraVars: [
                            war_file: 'target/*.war',
                            ansible_user: 'ec2-user',
                            ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
                        ],
                        credentialsId: 'n1a'
                    )
                }
            }
        }

        stage('Deploy on n2u') {
            steps {
                script {
                    unstash 'war'

                    ansiblePlaybook(
                        become: true,
                        inventory: 'hosts.ini',
                        playbook: 'javawebansible.yml',
                        extraVars: [
                            war_file: 'target/*.war',
                            ansible_user: 'ubuntu',
                            ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
                        ],
                        credentialsId: 'n2u'
                    )
                }
            }
        }

        stage('Deploy on n3c') {
            steps {
                script {
                    unstash 'war'

                    ansiblePlaybook(
                        become: true,
                        inventory: 'hosts.ini',
                        playbook: 'javawebansible.yml',
                        extraVars: [
                            war_file: 'target/*.war',
                            ansible_user: 'centos',
                            ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
                        ],
                        credentialsId: 'n3c'
                    )
                }
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
