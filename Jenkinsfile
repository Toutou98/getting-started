pipeline {
    agent {
        kubernetes {
            yaml """
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: maven
                image: maven:3.8.4-openjdk-11
                command:
                - cat
                tty: true
            """
        }
    }
    stages {
        stage('Checkout') {
            steps {
                container('maven') {
                    dir('/'){
                        // Checkout the code from the repository
                        git branch: 'main', url: 'https://github.com/Toutou98/getting-started.git'
                        sh 'pwd'
                    }
                }
            }
        }
        stage('Build') {
            steps {
                container('maven') {
                    dir('/getting-started') { // Ensure you're in the correct directory
                        // List files and check permissions
                        sh 'pwd'
                        sh 'ls -la'
                        // Set executable permissions for the Maven Wrapper
                        sh 'chmod +x mvnw'
                        // Build the Quarkus project using Maven Wrapper
                        sh './mvnw package -Dquarkus.package.jar.type=uber-jar'
                    }
                }
            }
        }
        stage('Archive') {
            steps {
                // Archive the JAR file
                archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
            }
        }
    }
    post {
        always {
            // Clean up workspace
            cleanWs()
        }
    }
}
