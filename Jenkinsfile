pipeline {
    agent none

    environment {
        WORKSPACE_DIR = pwd()
        TOMCAT_WEBAPPS_DIR = '/usr/local/bin/apache-tomcat-10.1.16/webapps'
    }

    stages {
        stage('Build and Test') {
            agent { label 'n4c' }
            steps {
                echo 'Building the project and running tests using N4C'
                script {
                    sh 'mvn clean package'
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
                            sh "cp -r ${WORKSPACE_DIR}/target/*.war ${TOMCAT_WEBAPPS_DIR}/"
                        }
                    }
                }
                stage('Node 2U') {
                    agent { label 'n2u' }
                    steps {
                        script {
                            echo 'Copying to Node 2U'
                            unstash 'build'
                            sh "cp -r ${WORKSPACE_DIR}/target/*.war ${TOMCAT_WEBAPPS_DIR}/"
                        }
                    }
                }
                stage('Node 3C') {
                    agent { label 'n3c' }
                    steps {
                        script {
                            echo 'Copying to Node 3C'
                            unstash 'build'
                            sh "cp -r ${WORKSPACE_DIR}/target/*.war ${TOMCAT_WEBAPPS_DIR}/"
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
                        credentialsId: credentials('n6c')
                    )
                }
            }
        }

        stage('Clean Up and Diagnostic Output') {
            agent { label 'master' } // Run on Jenkins server
            steps {
                script {
                    echo 'Cleaning up unnecessary files or directories'
                    sh "rm -rf ${WORKSPACE_DIR}/target"
                    echo 'Current workspace contents:'
                    sh "ls -la ${WORKSPACE_DIR}"
                    echo 'Git log:'
                    sh 'git log -n 5'
                }
            }
        }
    }
}
