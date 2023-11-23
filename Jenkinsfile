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

        stage('Unstash on Ansible Master') {
            agent { label 'n6c' }
            steps {
                script {
                    echo 'Unstashing files on n6c'
                    unstash 'build'
                    script {
                        sh "cp \$(find ${WORKSPACE_DIR}/target -name '*.war') /tmp/your.war"
                    }
                }
            }
        }

        stage('Copy to Agents n1a, n2u, n3c') {
            agent { label 'n1a || n2u || n3c' }
            steps {
                script {
                    echo 'Copying .war file to n1a, n2u, n3c'
                    script {
                        if (env.NODE_NAME == 'n1a') {
                            sshagent(['n1a-ssh-credentials']) {
                                sh 'scp /tmp/*war ec2-user@n1a:/usr/local/bin/apache-tomcat-10.1.16/webapps'
                            }
                        } else if (env.NODE_NAME == 'n2u') {
                            sshagent(['n2u-ssh-credentials']) {
                                sh 'scp /tmp/*war ubuntu@n2u:/usr/local/bin/apache-tomcat-10.1.16/webapps'
                            }
                        } else if (env.NODE_NAME == 'n3c') {
                            sshagent(['n3c-ssh-credentials']) {
                                sh 'scp /tmp/*war centos@n3c:/usr/local/bin/apache-tomcat-10.1.16/webapps'
                            }
                        }
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
