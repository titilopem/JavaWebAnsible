pipeline {
    agent any

    environment {
        N4C_CREDENTIAL = credentials('n3c')
        N6C_CREDENTIAL = credentials('n3c')
        WORKSPACE_DIR = pwd()  // Setting WORKSPACE_DIR to Jenkins workspace
    }

    stages {
        stage('Checkout') {
            agent { label 'n4c' }
            steps {
                script {
                    checkout scm
                }
            }
        }

        stage('Build') {
            agent { label 'n4c' }
            steps {
                echo 'Building the project using N4C'
                script {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Test') {
            agent { label 'n4c' }
            steps {
                echo 'Running tests using N4C'
                script {
                    sh 'mvn test'
                    stash(name: 'build', includes: "target/*.war")
                }
            }
        }

        stage('Fetch Configuration Files') {
            agent { label 'n4c' }
            steps {
                script {
                    echo 'Fetching the latest configuration files from Git'
                    sh "rm -f ${WORKSPACE_DIR}/*.yml"
                    sh "rm -f ${WORKSPACE_DIR}/*.ini"
                    sh 'cp -f path/to/your/config/*.yml ${WORKSPACE_DIR}/'
                    sh 'cp -f path/to/your/config/*.ini ${WORKSPACE_DIR}/'
                }
            }
        }

        stage('Deploy on Ansible Master') {
            agent { label 'n6c' }
            steps {
                script {
                    echo 'Deploying on Ansible Master'
                    unstash 'build'
                    ansiblePlaybook(
                        playbook: 'javawebansible.yml',
                        inventory: 'hosts.ini',
                        extras: "--extra \"workspace=${WORKSPACE_DIR}\""
                    )
                }
            }
        }

        stage('Clean Up') {
            agent { any }
            steps {
                script {
                    echo 'Cleaning up unnecessary files or directories'
                    sh "rm -rf ${WORKSPACE_DIR}/target"
                }
            }
        }

        stage('Diagnostic Output') {
            agent { any }
            steps {
                script {
                    echo 'Current workspace contents:'
                    sh 'ls -la ${WORKSPACE_DIR}'
                    echo 'Git log:'
                    sh 'git log -n 5'
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded! Send success notification.'
            emailext (
                subject: "Success: ${currentBuild.fullDisplayName}",
                body: "Build, test, and deployment were successful. Congratulations!",
                to: 'olawalemada@gmail.com'
            )
        }
        failure {
            echo 'Pipeline failed! Send failure notification.'
            emailext (
                subject: "Failed: ${currentBuild.fullDisplayName}",
                body: "Something went wrong. Please check the build, test, and deployment logs.",
                to: 'olawalemada@gmail.com'
            )
        }
    }
}
