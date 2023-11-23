pipeline {
    agent any
    
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
                    echo 'Unstashing files on n1a'
                    unstash 'build'
                    sh """
                        sudo rm -f /usr/local/bin/apache-tomcat-10.1.16/webapps/*.war
                        war_file=\$(find \$(pwd)/target -name '*.war' -type f -print -quit)
                        if [ -n "\$war_file" ]; then
                           sudo mv "\$war_file" /usr/local/bin/apache-tomcat-10.1.16/webapps/
                        else
                           echo "No .war file found in the target directory."
                           exit 1
                        fi
                    """
                }
            }
        }

        stage('Unstash on n2u') {
            agent { label 'n2u' }
            steps {
                script {
                    echo 'Unstashing files on n2u'
                    unstash 'build'
                    sh """
                        sudo rm /usr/local/bin/apache-tomcat-10.1.16/webapps/*.war
                        sudo cp \$(find \$(pwd)/target -name '*.war') /usr/local/bin/apache-tomcat-10.1.16/webapps/
                    """
                }
            }
        }

        stage('Unstash on n3cc') {
            agent { label 'n3cc' }
            steps {
                script {
                    echo 'Unstashing files on n3cc'
                    unstash 'build'
                    sh """
                        sudo rm /usr/local/bin/apache-tomcat-10.1.16/webapps/*.war
                        sudo cp \$(find \$(pwd)/target -name '*.war') /usr/local/bin/apache-tomcat-10.1.16/webapps/
                    """
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
                    sh "rm -rf \$(pwd)/target"
                }
            }
        }

        stage('Diagnostic Output') {
            agent { label 'n6c' }
            steps {
                script {
                    echo 'Current workspace contents:'
                    sh 'ls -la $(pwd)'
                    echo 'Git log:'
                    sh 'git log -n 5'
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
                     body: "Build, test, and deployment were successful. Congratulations!"
            }
        }
        failure {
            echo 'Pipeline failed! Send failure notification.'
            script {
                mail to: 'olawalemada@gmail.com',
                     subject: "Failed: ${currentBuild.fullDisplayName}",
                     body: "Something went wrong. Please check the build, test, and deployment logs."
            }
        }
    }
}
