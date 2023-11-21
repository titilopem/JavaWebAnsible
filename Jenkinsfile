pipeline {
    agent none

    environment {
        WORKSPACE_PATH = '/home/centos/workspace/ansibleproject'
        CREDENTIALS_N1A = 'n1a'
        CREDENTIALS_N2U = 'n2u'
        CREDENTIALS_N3C = 'n3c'
    }

    stages {
        stage('Build and Test') {
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

        stage('Copy Web App to Nodes') {
            steps {
                script {
                    echo 'Copying the application to nodes'
                }
            }

            parallel {
                stage('Copy to n1a') {
                    agent {
                        label 'n1a'
                    }
                    steps {
                        script {
                            echo 'Copying the application to n1a'
                            unstash 'application'
                            unstash 'ansibleFiles'
                            // Your copy logic to n1a here
                        }
                    }
                }

                stage('Copy to n2u') {
                    agent {
                        label 'n2u'
                    }
                    steps {
                        script {
                            echo 'Copying the application to n2u'
                            unstash 'application'
                            unstash 'ansibleFiles'
                            // Your copy logic to n2u here
                        }
                    }
                }

                stage('Copy to n3c') {
                    agent {
                        label 'n3c'
                    }
                    steps {
                        script {
                            echo 'Copying the application to n3c'
                            unstash 'application'
                            unstash 'ansibleFiles'
                            // Your copy logic to n3c here
                        }
                    }
                }
            }
        }

        stage('Deploy with Ansible') {
            agent {
                label 'n4c'
            }
            steps {
                script {
                    sshagent(credentials: [CREDENTIALS_N1A, CREDENTIALS_N2U, CREDENTIALS_N3C]) {
                        unstash 'application'
                        unstash 'ansibleFiles'
                        ansiblePlaybook playbook: 'javawebansible.yml', inventory: 'hosts.ini'
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
