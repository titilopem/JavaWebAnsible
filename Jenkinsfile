pipeline {
    agent none
    stages {
        stage('Build and Test') {
            agent {
                label 'node4u'
            }
            steps {
                echo 'Building the application'
                // Define build steps here
                sh '/opt/maven/bin/mvn clean package'
                stash (name: 'projectansible', includes: ["target/*.war", "deploy_webapp.yml"])

                echo 'Running tests'
                // Define test steps here
                sh '/opt/maven/bin/mvn test'
            }
        }
        stage('Deploy') {
            agent none
            steps {
                echo 'Deploying the application'
                script {
                    // Use Ansible to deploy the application across three nodes
                    ansiblePlaybook playbook: 'javawebansible.yml', inventory: 'hosts.ini'
                }
            }
            parallel {
                stage('node1a') {
                    agent {
                        label 'node1a'
                    }
                    steps {
                        script {
                            unstash 'projectansible'
                            ansiblePlaybook playbook: 'javawebansible.yml', inventory: 'hosts.ini'
                        }
                    }
                }
                stage('node2u') {
                    agent {
                        label 'node2u'
                    }
                    steps {
                        script {
                            unstash 'projectansible'
                            ansiblePlaybook playbook: 'javawebansible.yml', inventory: 'hosts.ini'
                        }
                    }
                }
                stage('node3c') {
                    agent {
                        label 'node3c'
                    }
                    steps {
                        script {
                            unstash 'projectansible'
                            ansiblePlaybook playbook: 'javawebansible.yml', inventory: 'hosts.ini'
                        }
                    }
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
