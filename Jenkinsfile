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
                    stash(name: 'build', includes: "target/*.war")
                }
            }
        }

        stage('Deploy on Ansible Master') {
            agent { label 'n6c' }
            steps {
                script {
                    echo 'Deploying on Ansible Master'

                    // Unstash into n1a directory
                    unstash 'build'
                    dir('n1a') {
                        sh 'cp -r ~/workspace/ansibleproject/target/*.war .'
                    }

                    // Unstash into n2u directory
                    unstash 'build'
                    dir('n2u') {
                        sh 'cp -r ~/workspace/ansibleproject/target/*.war .'
                    }

                    // Unstash into n3c directory
                    unstash 'build'
                    dir('n3c') {
                        sh 'cp -r ~/workspace/ansibleproject/target/*.war .'
                    }

                    ansiblePlaybook(
                        playbook: 'javawebansible.yml',
                        inventory: 'hosts.ini',
                        extras: "--extra \"workspace=${WORKSPACE_DIR}\""
                    )
                }
            }
        }

        stage('Clean Up') {
            agent { label 'n6c' }
            steps {
                script {
                    echo 'Cleaning up unnecessary files or directories'

                    // Clean up n1a directory
                    sh "rm -rf ${WORKSPACE_DIR}/n1a"

                    // Clean up n2u directory
                    sh "rm -rf ${WORKSPACE_DIR}/n2u"

                    // Clean up n3c directory
                    sh "rm -rf ${WORKSPACE_DIR}/n3c"
                }
            }
        }

        stage('Diagnostic Output') {
            agent { label 'n6c' }
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
