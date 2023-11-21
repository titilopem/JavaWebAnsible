pipeline {
    agent none
    stages {
        stage('Build and Test') {
            agent {
                label 'node4u'
            }
            steps {
                echo 'Building the application'
                sh '/opt/maven/bin/mvn clean package'
                stash (name: 'projectansible', includes: ["target/*.war", "deploy_webapp.yml"])

                echo 'Running tests'
                sh '/opt/maven/bin/mvn test'
            }
        }
        stage('Deploy') {
            parallel {
                stage('node1a') {
                    agent {
                        label 'node1a'
                    }
                    steps {
                        echo 'Deploying the application to node1a'
                        script {
                            unstash 'projectansible'
                            ansiblePlaybook playbook: 'deploy_webapp.yml', inventory: 'inventory-node1.ini'
                        }
                    }
                }
                stage('node2u') {
                    agent {
                        label 'node2u'
                    }
                    steps {
                        echo 'Deploying the application to node2u'
                        script {
                            unstash 'projectansible'
                            ansiblePlaybook playbook: 'deploy_webapp.yml', inventory: 'inventory-node2.ini'
                        }
                    }
                }
                stage('node3c') {
                    agent {
                        label 'node3c'
                    }
                    steps {
                        echo 'Deploying the application to node3c'
                        script {
                            unstash 'projectansible'
                            ansiblePlaybook playbook: 'deploy_webapp.yml', inventory: 'inventory-node3.ini'
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
