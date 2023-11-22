pipeline {
    agent any

    environment {
        N4C_CREDENTIAL = credentials('n3c')
        N6C_CREDENTIAL = credentials('n3c')
        WORKSPACE_DIR = pwd()  // Setting WORKSPACE_DIR to Jenkins workspace
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Explicitly check out the code
                    checkout scm
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Building the project using N4C'
                script {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests using N4C'
                script {
                    sh 'mvn test'
                    stash(name: 'build', includes: "target/*.war")
                }
            }
        }

        stage('Fetch Configuration Files') {
            steps {
                script {
                    echo 'Fetching the latest configuration files from Git'
                    
                    // Clean up old .yml and .ini files
                    sh "rm -f ${WORKSPACE_DIR}/*.yml"
                    sh "rm -f ${WORKSPACE_DIR}/*.ini"

                    // Copy the latest .yml and .ini files to the workspace
                    sh 'cp -f path/to/your/config/*.yml ${WORKSPACE_DIR}/'
                    sh 'cp -f path/to/your/config/*.ini ${WORKSPACE_DIR}/'
                }
            }
        }

        stage('Deploy on Ansible Master') {
            steps {
                script {
                    echo 'Deploying on Ansible Master'

                    // Unstash the project files
                    unstash 'build'

                    // Deploy using Ansible playbook with WORKSPACE_DIR
                    ansiblePlaybook(
                        playbook: 'javawebansible.yml',
                        inventory: 'hosts.ini',
                        extras: "--extra \"workspace=${WORKSPACE_DIR}\""
                    )
                }
            }
        }

        stage('Clean Up') {
            steps {
                script {
                    echo 'Cleaning up unnecessary files or directories'

                    // Add your clean-up tasks here, for example:
                    sh "rm -rf ${WORKSPACE_DIR}/target"
                }
            }
        }

        stage('Diagnostic Output') {
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
