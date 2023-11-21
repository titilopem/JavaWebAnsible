pipeline {
    agent none
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
        stage('Deploy') {
            agent {
                label 'n1a || n2u || n3c'
            }
            steps {
                echo 'Deploying the application'
                script {
                    unstash 'ansibleproject'
                    sh 'ansible-playbook -i hosts.ini javawebansible.yml'
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
