pipeline {
    agent any

    environment {
        N4C_CREDENTIAL = credentials('n3c')
        N6C_CREDENTIAL = credentials('n3c')
        N2U_CREDENTIAL = credentials('n2u')
        N1A_CREDENTIAL = credentials('n1a')
        
        WORKSPACE_DIR = env.WORKSPACE  // Setting WORKSPACE_DIR to Jenkins workspace
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

        stage('Unstash on n1a') {
            agent { label 'n1a' }
            steps {
                script {
                    echo "Listing files in ${WORKSPACE_DIR}/target:"
                    sh "ls -la ${WORKSPACE_DIR}/target"
                    echo "Finding war files:"
                    sh "find ${WORKSPACE_DIR}/target -name '*.war'"
                    echo 'Unstashing files on n1a'
                    unstash 'build'
                    echo 'Copying war files:'
                    sh "cp \$(find ${WORKSPACE_DIR}/target -name '*.war') /usr/local/bin/apache-tomcat-10.1.16/webapps/"
                    }
                }
            }
        }

        stage('Unstash on n2u') {
            agent { label 'n2u' }
            steps {
                script {
                    echo 'Unstashing files on n2u'
                    unstash 'build'
                    script {
                        sh "cp \$(find ${WORKSPACE_DIR}/target -name '*.war') /usr/local/bin/apache-tomcat-10.1.16/webapps/"
                    }
                }
            }
        }
        stage('Unstash on n3c') {
            agent { label 'n3c' }
            steps {
                script {
                    echo 'Unstashing files on n3c'
                    unstash 'build'
                    script {
                        sh "cp \$(find ${WORKSPACE_DIR}/target -name '*.war') /usr/local/bin/apache-tomcat-10.1.16/webapps/"
                    }
                }
            }
        }

        stage('Deploy with Ansible Master') {
            agent { label 'n6c' }
            steps {
                script {
                    ansiblePlaybook(
                        playbook: 'javawebansible.yml',
                        inventory: 'hosts.ini'
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
