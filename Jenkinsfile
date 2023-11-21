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
                echo 'Fetching Ansible files from GitHub'
                dir(WORKSPACE_PATH) {
                    checkout scm
                }
            }
        }
        stage('Build') {
            agent {
                label 'n4c'
            }
            steps {
                echo 'Building the application'
                sh '/opt/maven/bin/mvn clean package'
            }
        }
        stage('Test') {
            agent {
                label 'n4c'
            }
            steps {
                echo 'Running tests'
                sh 'mvn test'
                stash(name: 'ansibleproject', includes: "${WORKSPACE_PATH}/target/*.war")
            }
        }
        stage('Deploy on Amazon') {
            agent {
                label 'n1a'
            }
            steps {
                echo 'Deploying the application to n1a'
                script {
                    unstash 'ansibleproject'
                    ansiblePlaybook playbook: "${WORKSPACE_PATH}/javawebansible.yml", inventory: "${WORKSPACE_PATH}/hosts.ini"
                }
            }
        }
        stage('Deploy on Ubuntu') {
            agent {
                label 'n2u'
            }
            steps {
                echo 'Deploying the application to n2u'
                script {
                    unstash 'ansibleproject'
                    ansiblePlaybook playbook: "${WORKSPACE_PATH}/javawebansible.yml", inventory: "${WORKSPACE_PATH}/hosts.ini"
                }
            }
        }
        stage('Deploy on CentOS') {
            agent {
                label 'n3c'
            }
            steps {
                echo 'Deploying the application to n3c'
                script {
                    unstash 'ansibleproject'
                    ansiblePlaybook playbook: "${WORKSPACE_PATH}/javawebansible.yml", inventory: "${WORKSPACE_PATH}/hosts.ini"
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
