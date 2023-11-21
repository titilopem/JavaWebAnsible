pipeline {
    agent none
    environment {
        SSH_KEY_N1A = credentials('n1a')  // Replace with your SSH key credential ID for n1a
        SSH_KEY_N2U = credentials('n2u')  // Replace with your SSH key credential ID for n2u
        SSH_KEY_N3C = credentials('n3c')  // Replace with your SSH key credential ID for n3c
        TOMCAT_USER_N1A = 'ec2-user'  // Update with the appropriate Tomcat user for n1a
        TOMCAT_USER_N2U = 'ubuntu'    // Update with the appropriate Tomcat user for n2u
        TOMCAT_USER_N3C = 'centos'    // Update with the appropriate Tomcat user for n3c
    }
    stages {
        stage('Checkout') {
            agent any
            steps {
                checkout scm
            }
        }

        stage('Build on n4c') {
            agent {
                label 'n4c'
            }
            steps {
                script {
                    // Your build steps
                    sh 'mvn clean package'
                    stash(name: 'war', includes: 'target/*.war')
                }
            }
        }

        stage('Test on n4c') {
            agent {
                label 'n4c'
            }
            steps {
                script {
                    // Your test steps (e.g., mvn test)
                    sh 'mvn test'
                }
            }
        }

        stage('Deploy') {
            agent any
            parallel {
                stage('Deploy on n1a') {
                    steps {
                        script {
                            unstash 'war'
                            // Your deployment steps for n1a
                            ansiblePlaybook(
                 
