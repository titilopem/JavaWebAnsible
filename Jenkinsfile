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
                sh '/opt/maven/bin/mvn test'
                stash (name: 'projectansible', includes: "target/*.war")
            }
        }
        stage('Deploy') {
            agent {
                label 'node2u || node1a || node3c'
            }
            steps {
                echo 'Deploying the application'
                // Define deployment steps here
                unstash 'projectansible'
                script {
                    def apacheDir = "~/apache-tomcat/webapps/"
                    def tomcatBin = "~/apache-tomcat/bin/"

                    // Adjust commands based on the operating system
                    if (isUnix()) {
                        sh "sudo rm -rf ${apacheDir}/*.war"
                        sh "sudo mkdir -p ${apacheDir}"  // Create the directory if it doesn't exist
                        sh "sudo mv target/*.war ${apacheDir}"
                        sh "sudo systemctl daemon-reload"
                        sh "${tomcatBin}/startup.sh"
                    } else if (isWindows()) {
                        // Add Windows deployment commands if needed
                        echo "Windows deployment steps go here"
                    } else {
                        // Add other operating systems deployment commands if needed
                        echo "Other OS deployment steps go here"
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
