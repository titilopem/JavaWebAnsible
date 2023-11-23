pipeline {
    agent none

    environment {
        WORKSPACE_DIR = ''
        TOMCAT_WEBAPPS_DIR = '/usr/local/bin/apache-tomcat-10.1.16/webapps'
        // Add any other environment variables here
    }

    parameters {
        // You can add parameters for node labels if needed
        string(name: 'NODE_1A_LABEL', defaultValue: 'n1a', description: 'Label for Node 1A')
        string(name: 'NODE_2U_LABEL', defaultValue: 'n2u', description: 'Label for Node 2U')
        string(name: 'NODE_3C_LABEL', defaultValue: 'n3c', description: 'Label for Node 3C')
        string(name: 'NODE_6C_LABEL', defaultValue: 'n6c', description: 'Label for Node 6C')
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
                            deployToNode(params.NODE_1A_LABEL)
                        }
                    }
                }
                stage('Node 2U') {
                    steps {
                        script {
                            echo 'Copying to Node 2U'
                            deployToNode(params.NODE_2U_LABEL)
                        }
                    }
                }
                stage('Node 3C') {
                    steps {
                        script {
                            echo 'Copying to Node 3C'
                            deployToNode(params.NODE_3C_LABEL)
                        }
                    }
                }
            }
        }

        stage('Deploy on Node 6C') {
            agent { label params.NODE_6C_LABEL }
            steps {
                script {
                    echo "Deploying on Node ${params.NODE_6C_LABEL}"
                    deployToNode(params.NODE_6C_LABEL)
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
        return {
            node(nodeLabel) {
                script {
                    unstash 'build'
                    sh "cp -r ${WORKSPACE_DIR}/target/*.war ${TOMCAT_WEBAPPS_DIR}/"
                }
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
