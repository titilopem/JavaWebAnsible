pipeline {
    agent {
        label 'n4c'
    }

    environment {
        WORKSPACE_PATH = '/home/centos/workspace/ansibleproject'
    }

    stages {
        stage('Build and Test') {
            steps {
                dir(WORKSPACE_PATH) {
                    // Build and test your web application on n4c
                    sh '/opt/maven/bin/mvn clean package'
                    stash name: 'application', includes: 'target/**/*.war'
                }
            }
        }

        stage('Fetch Ansible Files') {
            steps {
                dir(WORKSPACE_PATH) {
                    // Fetch and stash Ansible-related files from Git
                    checkout scm
                    stash name: 'ansibleFiles', includes: 'javawebansible.yml, hosts.ini'
                }
            }
        }

        stage('Deploy to Nodes') {
            parallel {
                stage('Deploy to n1a') {
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
                stage('Deploy to n2u') {
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
                stage('Deploy to n3c') {
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
