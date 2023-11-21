pipeline {
    agent none
    environment {
        PATH = "/usr/bin:$PATH"
    }
    stages {
        stage('Build') {
            agent {
                label 'n4c'
            }
            steps {
                echo 'Building the application'
                // Define build steps here
                sh '/opt/maven/bin/mvn clean package'
            }
        }
        stage('Test') {
            agent {
                label 'n4c'
            }
            steps {
                echo 'Running tests'
                // Define test steps here
                sh 'mvn test'
                stash (name: 'ansibleproject', includes: "target/*.war")
            }
        }
        stage('Deploy on Amazon') {
            agent {
                label 'n1a'
            }
            steps {
                echo 'Deploying the application to n1a'
                script {
                    unstash 'ansibleproject'
                    ansiblePlaybook playbook: 'javawebansible.yml', inventory: 'hosts.ini'
                }
            }
        }
        stage('Deploy on Ubuntu') {
            agent {
                label 'n2u'
            }
            steps {
                echo 'Deploying the application to n2u'
                script {
                    unstash 'ansibleproject'
                    ansiblePlaybook playbook: 'javawebansible.yml', inventory: 'hosts.ini'
                }
            }
        }
        stage('Deploy on Centos') {
            agent {
                label 'n3c'
            }
            steps {
                echo 'Deploying the application to n3c'
                script {
                    unstash 'ansibleproject'
                    ansiblePlaybook playbook: 'javawebansible.yml', inventory: 'hosts.ini'
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline succeeded! Send success notification.'
            script {
                mail to: 'olawalemada@gmail.com',
                subject: "Success: ${currentBuild.fullDisplayName}",
                body: "Build and deployment were successful. Congratulations!"
            }
        }
        failure {
            echo 'Pipeline failed! Send failure notification.'
            script {
                mail to: 'olawalemada@gmail.com',
                subject: "Failed: ${currentBuild.fullDisplayName}",
                body: "Something went wrong. Please check the build and deployment logs."
            }
        }
    }
}
