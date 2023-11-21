pipeline {
    agent none

    environment {
        WORKSPACE_PATH = '/home/centos/workspace/ansibleproject'
        CREDENTIALS_N1A = 'n1a'
        CREDENTIALS_N2U = 'n2u'
        CREDENTIALS_N3C = 'n3c'
    }

    stages {
        stage('Build') {
            agent {
                label 'n4c'
            }
            steps {
                dir(WORKSPACE_PATH) {
                    checkout scm
                    sh '/opt/maven/bin/mvn clean package'
                    stash name: 'application', includes: 'target/**/*.war'
                    stash name: 'ansibleFiles', includes: '*.yml, *.ini'
                }
            }
        }

        stage('Test') {
            agent {
                label 'n4c'
            }
            steps {
                dir(WORKSPACE_PATH) {
                    sh '/opt/maven/bin/mvn test'
                }
            }
        }

        stage('Deploy on n1a') {
            agent {
                label 'n1a'
            }
            steps {
                script {
                    sshagent(credentials: [CREDENTIALS_N1A]) {
                        unstash 'application'
                        unstash 'ansibleFiles'
                        ansiblePlaybook playbook: 'javawebansible.yml', inventory: 'hosts.ini', extraVars: [target_node: 'n1a']
                    }
                }
            }
        }

        stage('Deploy on n2u') {
            agent {
                label 'n2u'
            }
            steps {
                script {
                    sshagent(credentials: [CREDENTIALS_N2U]) {
                        unstash 'application'
                        unstash 'ansibleFiles'
                        ansiblePlaybook playbook: 'javawebansible.yml', inventory: 'hosts.ini', extraVars: [target_node: 'n2u']
                    }
                }
            }
        }

        stage('Deploy on n3c') {
            agent {
                label 'n3c'
            }
            steps {
                script {
                    sshagent(credentials: [CREDENTIALS_N3C]) {
                        unstash 'application'
                        unstash 'ansibleFiles'
                        ansiblePlaybook playbook: 'javawebansible.yml', inventory: 'hosts.ini', extraVars: [target_node: 'n3c']
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
