pipeline {
    agent none

    environment {
        WORKSPACE_PATH = '/home/centos/workspace/ansibleproject'
    }

    stages {
        stage('Fetch Ansible Files') {
            agent {
                label 'n4c'
            }
            steps {
                dir(WORKSPACE_PATH) {
                    // Fetch and stash Ansible-related files
                    checkout scm
                    stash name: 'ansibleFiles', includes: ['*.yml', '*.ini']
                }
            }
        }

        stage('Build') {
            agent {
                label 'n4c'
            }
            steps {
                dir(WORKSPACE_PATH) {
                    sh '/opt/maven/bin/mvn clean package'
                    stash name: 'webApp', includes: 'target/**/*.war'
                }
            }
        }

        stage('Test') {
            agent {
                label 'n4c'
            }
            steps {
                dir(WORKSPACE_PATH) {
                    sh 'mvn test'
                }
            }
        }

        stage('Deploy on Amazon') {
            agent {
                label 'n1a'
            }
            steps {
                script {
                    WORKSPACE_PATH = '/home/ec2-user/workspace/ansibleproject'
                    echo 'Deploying the application to n1a'
                    unstash 'ansibleFiles'  // Unstash Ansible-related files
                    sh '/usr/bin/ansible-playbook -i inventory.ini javawebansible.yml'
                    unstash 'webApp'  // Unstash web application
                    // Copy .war file to destination on n1a
                }
            }
        }

        stage('Deploy on Ubuntu') {
            agent {
                label 'n2u'
            }
            steps {
                script {
                    WORKSPACE_PATH = '/home/ubuntu/workspace/ansibleproject'
                    echo 'Deploying the application to n2u'
                    unstash 'ansibleFiles'  // Unstash Ansible-related files
                    sh '/usr/bin/ansible-playbook -i inventory.ini javawebansible.yml'
                    unstash 'webApp'  // Unstash web application
                    // Copy .war file to destination on n2u
                }
            }
        }

        stage('Deploy on CentOS') {
            agent {
                label 'n3c'
            }
            steps {
                script {
                    WORKSPACE_PATH = '/home/centos/workspace/ansibleproject'
                    echo 'Deploying the application to n3c'
                    unstash 'ansibleFiles'  // Unstash Ansible-related files
                    sh '/usr/bin/ansible-playbook -i inventory.ini javawebansible.yml'
                    unstash 'webApp'  // Unstash web application
                    // Copy .war file to destination on n3c
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
                     body: "Build and deployment were successful. Congratulations!"
            }
        }
        failure {
            echo 'Pipeline failed! Send failure notification.'
            script {
                mail to: 'olawalemada@gmail.com',
                     subject: "Failed: ${currentBuild.fullDisplayName}",
                     body: "Something went wrong. Please check the build and deployment logs."
            }
        }
    }
}
