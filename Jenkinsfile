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
                // Checkout the code from the repository
                git branch: 'main', url: 'https://github.com/Toutou98/getting-started.git'
            }
        }
        stage('Build') {
            steps {
                container('maven') {
                    dir('getting-started') {
                        // Set executable permissions for the Maven Wrapper
                        sh 'chmod +x /getting-started/mvnw'
                        // Build the Quarkus project using Maven Wrapper
                        sh './getting-started/mvnw package -Dquarkus.package.jar.type=uber-jar'
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
