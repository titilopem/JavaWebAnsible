pipeline {
    agent any

    environment {
        N1A_CREDENTIAL = credentials('n1a')
        N2C_CREDENTIAL = credentials('n2c')
        N3C_CREDENTIAL = credentials('n3c')
        N6C_CREDENTIAL = credentials('n6c')  // Corrected typo
        WORKSPACE_DIR = pwd()  // Setting WORKSPACE_DIR to Jenkins workspace
    }

    stages {
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
                    stash(name: 'build', includes: 'target/*.war')
                }
            }
        }

        stage('Deploy on Nodes') {
            parallel {
                stage('Node 1A') {
                    agent { label 'n1a' }
                    steps {
                        script {
                            echo 'Copying to Node 1A'
                            unstash 'build'
                            // Add steps for copying to Node 1A
                        }
                    }
                }
                stage('Node 2C') {
                    agent { label 'n2c' }
                    steps {
                        script {
                            echo 'Copying to Node 2C'
                            unstash 'build'
                            // Add steps for copying to Node 2C
                        }
                    }
                }
                stage('Node 3C') {
                    agent { label 'n3c' }
                    steps {
                        script {
                            echo 'Copying to Node 3C'
                            unstash 'build'
                            // Add steps for copying to Node 3C
                        }
                    }
                }
            }
        }

        stage('Deploy on N6C') {
            agent { label 'n6c' }
            steps {
                script {
                    echo 'Deploying on Node 6C'
                    ansiblePlaybook(
                        playbook: 'javawebansible.yml',
                        inventory: 'hosts.ini',
                        credentialsId: "${N6C_CREDENTIAL}"
                    )
                }
            }
        }

        stage('Clean Up') {
            agent { label 'n6c' }
            steps {
                script {
                    echo 'Cleaning up unnecessary files or directories'
                    sh "rm -rf ${WORKSPACE_DIR}/target"
                }
            }
        }

        stage('Diagnostic Output') {
            agent { label 'n6c' }
            steps {
                script {
                    echo 'Current workspace contents:'
                    sh "ls -la ${WORKSPACE_DIR}"
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
