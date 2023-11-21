pipeline {
    agent none

    environment {
        CREDENTIALS_N4C = 'n4c'
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
                dir('/home/centos/workspace/ansibleproject') {
                    checkout scm
                    sh '/opt/maven/bin/mvn clean package'
                    stash name: 'ansibleproject', includes: 'target/*.war'
                }
            }
        }

        stage('Test') {
            agent {
                label 'n4c'
            }
            steps {
                dir('/home/centos/workspace/ansibleproject') {
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
                        echo 'Deploying the application'
                        unstash 'ansibleproject'
                        ansiblePlaybook playbook: 'javawebansible.yml', inventory: 'hosts.ini', extraVars: [workspace_path: '/home/centos/workspace/ansibleproject']
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
                        echo 'Deploying the application'
                        unstash 'ansibleproject'
                        ansiblePlaybook playbook: 'javawebansible.yml', inventory: 'hosts.ini', extraVars: [workspace_path: '/home/centos/workspace/ansibleproject']
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
                        echo 'Deploying the application'
                        unstash 'ansibleproject'
                        ansiblePlaybook playbook: 'javawebansible.yml', inventory: 'hosts.ini', extraVars: [workspace_path: '/home/centos/workspace/ansibleproject']
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
