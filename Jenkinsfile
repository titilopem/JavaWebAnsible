pipeline {
    agent none
    environment {
        WORKSPACE_PATH = ''
    }
    stages {
        stage('Build') {
            agent {
                label 'n4c'
            }
            steps {
                script {
                    WORKSPACE_PATH = '/home/centos/workspace/ansibleproject'
                    echo 'Building the application'
                    sh '/opt/maven/bin/mvn clean package'
                }
            }
        }
        stage('Test') {
            agent {
                label 'n4c'
            }
            steps {
                script {
                    echo 'Running tests'
                    sh 'mvn test'
                    stash (name: 'ansibleproject', includes: "${WORKSPACE_PATH}/target/*.war")
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
                script {
                    WORKSPACE_PATH = '/home/ubuntu/workspace/ansibleproject' // Adjust the path
                    echo 'Deploying the application to n2u'
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
                script {
                    WORKSPACE_PATH = '/home/centos/workspace/ansibleproject' // Adjust the path
                    echo 'Deploying the application to n3c'
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
