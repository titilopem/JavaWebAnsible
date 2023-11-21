pipeline {
    agent none

    environment {
        WORKSPACE_PATH = '/home/centos/workspace/ansibleproject'
        CREDENTIALS_N1A = 'n1a'
        CREDENTIALS_N2U = 'n2u'
        CREDENTIALS_N3C = 'n3c'
    }

    stages {
        stage('Start SSH Agent for n1a') {
            agent any
            steps {
                sshagent(credentials: [CREDENTIALS_N1A]) {
                    sh 'env'
                }
            }
        }

        stage('Fetch Ansible Files') {
            agent {
                label 'n4c'
            }
            steps {
                dir(WORKSPACE_PATH) {
                    checkout scm
                    stash name: 'ansibleFiles', includes: '*.yml, *.ini'
                }
            }
        }

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
                            ansiblePlaybook playbook: 'javawebansible.yml', inventory: 'hosts.ini', extraVars: [target_node: 'n1a']
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
                            ansiblePlaybook playbook: 'javawebansible.yml', inventory: 'hosts.ini', extraVars: [target_node: 'n2u']
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
                            ansiblePlaybook playbook: 'javawebansible.yml', inventory: 'hosts.ini', extraVars: [target_node: 'n3c']
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
