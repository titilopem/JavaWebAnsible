pipeline {
    agent none
    environment {
        SSH_KEY_N1A = credentials('n1a')  // Replace with your SSH key credential ID for n1a
        SSH_KEY_N2U = credentials('n2u')  // Replace with your SSH key credential ID for n2u
        SSH_KEY_N3C = credentials('n3c')  // Replace with your SSH key credential ID for n3c
        TOMCAT_USER_N1A = 'ec2-user'  // Update with the appropriate Tomcat user for n1a
        TOMCAT_USER_N2U = 'ubuntu'    // Update with the appropriate Tomcat user for n2u
        TOMCAT_USER_N3C = 'centos'    // Update with the appropriate Tomcat user for n3c
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
                    // Your build and test steps
                    sh 'mvn clean package'
                    stash(name: 'war', includes: 'target/*.war')
                }
            }
        }

        stage('Deploy on n1a') {
            agent {
                label 'n1a'
            }
            steps {
                node('n1a') {
                    script {
                        unstash 'war'
                        // Your deployment steps
                        ansiblePlaybook(
                            become: true,
                            inventory: 'hosts.ini',
                            playbook: 'javawebansible.yml',
                            extraVars: [
                                war_file: 'target/*.war',
                                ansible_user: env.TOMCAT_USER_N1A,
                                ansible_ssh_private_key_file: env.SSH_KEY_N1A,
                                ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
                            ]
                        )
                    }
                }
            }
        }

        stage('Deploy on n2u') {
            agent {
                label 'n2u'
            }
            steps {
                node('n2u') {
                    script {
                        unstash 'war'
                        // Your deployment steps
                        ansiblePlaybook(
                            become: true,
                            inventory: 'hosts.ini',
                            playbook: 'javawebansible.yml',
                            extraVars: [
                                war_file: 'target/*.war',
                                ansible_user: env.TOMCAT_USER_N2U,
                                ansible_ssh_private_key_file: env.SSH_KEY_N2U,
                                ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
                            ]
                        )
                    }
                }
            }
        }

        stage('Deploy on n3c') {
            agent {
                label 'n3c'
            }
            steps {
                node('n3c') {
                    script {
                        unstash 'war'
                        // Your deployment steps
                        ansiblePlaybook(
                            become: true,
                            inventory: 'hosts.ini',
                            playbook: 'javawebansible.yml',
                            extraVars: [
                                war_file: 'target/*.war',
                                ansible_user: env.TOMCAT_USER_N3C,
                                ansible_ssh_private_key_file: env.SSH_KEY_N3C,
                                ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
                            ]
                        )
                    }
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    deleteDir()
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
