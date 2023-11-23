pipeline {
    agent none

    environment {
        WORKSPACE_DIR = ''
        TOMCAT_WEBAPPS_DIR = '/usr/local/bin/apache-tomcat-10.1.16/webapps'
    }

    stages {
        stage('Build and Test') {
            agent { label 'n4c' }
            steps {
                script {
                    echo 'Building the project and running tests using N4C'
                    dir(WORKSPACE_DIR) {
                        sh 'mvn clean package'
                        sh 'mvn test'
                        stash(name: 'build', includes: 'target/*.war')
                    }
                }
            }
        }

        stage('Deploy on Nodes') {
            parallel {
                stage('Node 1A') {
                    steps {
                        script {
                            echo 'Copying to Node 1A'
                            deployToNode('n1a')
                        }
                    }
                }
                stage('Node 2U') {
                    steps {
                        script {
                            echo 'Copying to Node 2U'
                            deployToNode('n2u')
                        }
                    }
                }
                stage('Node 3C') {
                    steps {
                        script {
                            echo 'Copying to Node 3C'
                            deployToNode('n3c')
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
                    deployToNode('n6c')
                }
            }
        }

        stage('Clean Up and Diagnostic Output') {
            agent { label 'master' }
            steps {
                script {
                    echo 'Cleaning up unnecessary files or directories'
                    cleanUpWorkspace()
                    displayWorkspaceContents()
                    displayGitLog()
                }
            }
        }
    }

    def deployToNode(nodeLabel) {
        node(nodeLabel) {
            script {
                unstash 'build'
                sh "cp -r ${WORKSPACE_DIR}/target/*.war ${TOMCAT_WEBAPPS_DIR}/"
            }
        }
    }

    def cleanUpWorkspace() {
        sh "rm -rf ${WORKSPACE_DIR}/target"
    }

    def displayWorkspaceContents() {
        echo 'Current workspace contents:'
        sh "ls -la ${WORKSPACE_DIR}"
    }

    def displayGitLog() {
        echo 'Git log:'
        sh 'git log -n 5'
    }
}
