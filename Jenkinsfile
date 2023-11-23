pipeline {
    agent none

    environment {
        N4C_CREDENTIAL = credentials('n4c')
        N6C_CREDENTIAL = credentials('n6c')
        WORKSPACE_DIR = env.WORKSPACE
        TOMCAT_WEBAPPS_DIR = '/path/to/tomcat/webapps/'
        SSH_USER_N1A = 'ec2-user'
        SSH_USER_N2U = 'ubuntu'
        SSH_USER_N3C = 'centos'
    }

    stages {
        stage('Checkout and Build') {
            agent { label 'n4c' }
            steps {
                script {
                    checkout scm
                    echo 'Building the project using N4C'
                    sh 'mvn clean package'
                    stash(name: 'build', includes: "target/*.war")
                }
            }
        }

        stage('Deploy on Nodes') {
            agent { label 'n6c' }
            steps {
                script {
                    echo 'Deploying on Ansible Master'

                    // Unstash the .war file
                    unstash 'build'

                    // Copy .war file to n1a (Amazon) directory
                    sh "scp ${WORKSPACE_DIR}/target/*.war ${SSH_USER_N1A}@n1a:${TOMCAT_WEBAPPS_DIR}"

                    // Copy .war file to n2u (Ubuntu) directory
                    sh "scp ${WORKSPACE_DIR}/target/*.war ${SSH_USER_N2U}@n2u:${TOMCAT_WEBAPPS_DIR}"

                    // Copy .war file to n3c (CentOS) directory
                    sh "scp ${WORKSPACE_DIR}/target/*.war ${SSH_USER_N3C}@n3c:${TOMCAT_WEBAPPS_DIR}"

                    ansiblePlaybook(
                        playbook: 'javawebansible.yml',
                        inventory: 'hosts.ini',
                        extras: "--extra \"workspace=${WORKSPACE_DIR}\""
                    )
                }
            }
        }
    }
}
