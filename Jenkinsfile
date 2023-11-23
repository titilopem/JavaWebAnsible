pipeline {
    agent none

    environment {
        N1A_CREDENTIAL = credentials('n1a')
        N2C_CREDENTIAL = credentials('n2c')
        N3C_CREDENTIAL = credentials('n3c')
        N6C_CREDENTIAL = credentials('n3c')
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
                    stash(name: 'build', includes: "target/*.war")
                }
            }
        }

        stage('Deploy on Nodes') {
            parallel {
                stage('Node 1A') {
                    agent { label 'n1a' }
                    steps {
                        script {
                            echo 'Deploying on Node 1A'
                            unstash 'build'
                            ansiblePlaybook(
                                playbook: 'javawebansible.yml',
                                inventory: 'hosts.ini',
                                extras: "--extra \"workspace=${WORKSPACE_DIR}\" dest=/usr/local/bin/apache-tomcat-10.1.16/webapps/",
                                credentialsId: "${N1A_CREDENTIAL}"
                            )
                        }
                    }
                }
                stage('Node 2C') {
                    agent { label 'n2c' }
                    steps {
                        script {
                            echo 'Deploying on Node 2C'
                            unstash 'build'
                            ansiblePlaybook(
                                playbook: 'javawebansible.yml',
                                inventory: 'hosts.ini',
                                extras: "--extra \"workspace=${WORKSPACE_DIR}\" dest=/usr/local/bin/apache-tomcat-10.1.16/webapps/",
                                credentialsId: "${N2C_CREDENTIAL}"
                            )
                        }
                    }
                }
                stage('Node 3C') {
                    agent { label 'n3c' }
                    steps {
                        script {
                            echo 'Deploying on Node 3C'
                            unstash 'build'
                            ansiblePlaybook(
                                playbook: 'javawebansible.yml',
                                inventory: 'hosts.ini',
                                extras: "--extra \"workspace=${WORKSPACE_DIR}\" dest=/usr/local/bin/apache-tomcat-10.1.16/webapps/",
                                credentialsId: "${N3C_CREDENTIAL}"
                            )
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
                    unstash 'build'
                    ansiblePlaybook(
                        playbook: 'javawebansible.yml',
                        inventory: 'hosts.ini',
                        extras: "--extra \"workspace=${WORKSPACE_DIR}\" dest=/usr/local/bin/apache-tomcat-10.1.16/webapps/",
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
