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
                    stash name: 'ansibleFiles', includes: '*.yml, *.ini'
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
                    stash name: 'application', includes: 'target/**/*.war'
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

        stage('Deploy on Nodes') {
            parallel {
                stage('Deploy on Amazon') {
                    agent {
                        label 'n1a'
                    }
                    steps {
                        script {
                            echo 'Deploying the application to n1a'
                            unstash 'application'
                            unstash 'ansibleFiles'
                            ansiblePlaybook playbook: "${WORKSPACE_PATH}/javawebansible.yml", inventory: "${WORKSPACE_PATH}/hosts.ini"
                        }
                    }
                }

                stage('Deploy on Ubuntu') {
                    agent {
                        label 'n2u'
                    }
                    steps {
                        script {
                            echo 'Deploying the application to n2u'
                            unstash 'application'
                            unstash 'ansibleFiles'
                            ansiblePlaybook playbook: "${WORKSPACE_PATH}/javawebansible.yml", inventory: "${WORKSPACE_PATH}/hosts.ini"
                        }
                    }
                }

                stage('Deploy on CentOS') {
                    agent {
                        label 'n3c'
                    }
                    steps {
                        script {
                            echo 'Deploying the application to n3c'
                            unstash 'application'
                            unstash 'ansibleFiles'
                            ansiblePlaybook playbook: "${WORKSPACE_PATH}/javawebansible.yml", inventory: "${WORKSPACE_PATH}/hosts.ini"
                        }
                    }
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
