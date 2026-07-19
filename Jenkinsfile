/* pipeline {
    agent any
    tools {
	jdk 'Java21'
	maven 'maven'
    }
    stages {
        stage('Clean') {
            steps {
                echo 'Cleaning...'
                sh 'mvn clean'
            }
        }
        stage('Compile') {
            steps {
                echo 'Compileing...'
                sh 'mvn compile'
            }
        }
        stage('Package') {
            steps {
                echo 'Package...'
                sh 'mvn package'
            }
        }
	 stage('Deploy') {
            steps {
                echo 'Deploying...'
                sh 'mvn install tomcat7:deploy'
            }
        }

    }
}*/
pipeline {
    agent none // Defined at stage level for multi-agent execution
	tools {
	jdk 'Java21'
	maven 'maven'
    }

    environment {
        TOMCAT_URL = 'http://54.161.22.157:8080/manager/html'
    }

    stages {
        // --- AGENT 1 TASKS: Checkout, Compile, Package ---
        stage('Source Code Checkout') {
            agent { label 'agent1' }
            steps {
                cleanWs()
                checkout scm
            }
        }

        stage('Compile') {
            agent { label 'agent1' }
            steps {
                echo 'Compiling source code...'
                sh 'mvn clean compile' 
            }
        }

        stage('Package') {
            agent { label 'agent1' }
            steps {
                echo 'Packaging application into WAR file...'
                sh 'mvn package -DskipTests'
                // Stash the generated WAR file to transfer it to Agent-2
                stash name: 'war-artifact', includes: 'target/*.war'
            }
        }

        // --- AGENT 2 TASKS: Test and Deploy ---
        stage('Test') {
            agent { label 'agent2' }
            steps {
                // Checkout code on Agent 2 to run tests if needed, or unstash code
                checkout scm
                echo 'Running unit tests...'
                sh 'mvn test'
            }
        }

        stage('Deploy Application') {
            agent { label 'agent2' }
            steps {
                echo 'Deploying WAR file to Apache Tomcat...'
                // Retrieve the artifact created by Agent-1
                unstash 'war-artifact'
                
                withCredentials([usernamePassword(credentialsId: 'tomcat', passwordVariable: 'TOMCAT_PASS', usernameVariable: 'TOMCAT_USER')]) {
                    // Deploy via Tomcat text manager API using curl
                    sh """
                        curl -u $TOMCAT_USER:$TOMCAT_PASS -T target/*.war "${TOMCAT_URL}/deploy?path=/myapp&update=true"
                    """
                }
            }
        }
    }

    post {
        always {
            withCredentials([usernamePassword(credentialsId: 'smtp-creds', passwordVariable: 'SMTP_PASS', usernameVariable: 'SMTP_USER')]) {
                mail to: 'dsucharitha.99@gmail.com',
                     subject: "Jenkins Build ${currentBuild.fullDisplayName} - ${currentBuild.currentResult}",
                     body: """Job Name: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Build Status: ${currentBuild.currentResult}
Build Duration: ${currentBuild.durationString}
Build URL: ${env.BUILD_URL}""",
                     replyTo: 'no-reply@example.com',
                     from: "${SMTP_USER}"
            }
        }
    }
}

