pipeline {
    agent none
    stages {
        stage('Build') {
            agent {
                label 'node4u'
            }
            steps {
                echo 'Building the application'
                // Define build steps here
                sh '/opt/maven/bin/mvn clean package'
            }
        }
        stage('Test') {
            agent {
                label 'node4u'
            }
            steps {
                echo 'Running tests'
                // Define test steps here
                sh 'mvn test'
                stash (name: 'project-ansible', includes: "target/*.war")
            }
        }
        stage('Deploy') {
            agent {
                label 'node2u'
            }
            steps {
                echo 'Deploying the application'
                // Define deployment steps here
                unstash 'project-ansible'
                sh "sudo rm -rf ~/apache*/webapp/*.war" 
                sh "sudo mv target/*.war ~/apache*/webapps/"
                sh "sudo systemctl daemon-reload"
                sh "~/apache-tomcat-7.0.94/bin/startup.sh"
            }
        }
    }
    post {
        success {
            echo 'Pipeline succeeded! Send success notification.'
            script {
                mail to: 'olawalemada@gmail.com',
                    subject: "Success: ${currentBuild.fullDisplayName}",
                    body: "Build was successful. Congratulations!"
            }
        }
        failure {
            echo 'Pipeline failed! Send failure notification.'
            script {
                mail to: 'olawalemada@gmail.com',
                    subject: "Failed: ${currentBuild.fullDisplayName}",
                    body: "Something went wrong. Please check the build logs."
            }  
        }
    }
}
