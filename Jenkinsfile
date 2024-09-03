pipeline {
    agent {
        kubernetes {
            yaml """
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: maven
                image: maven:3.9.9-openjdk-17
                command:
                - cat
                tty: true
            """
        }
    }
    stages {
        stage('Build') {
            steps {
                container('maven') {
                    // List files and check permissions
                    git branch: 'main', url: 'https://github.com/Toutou98/getting-started.git'
                    sh 'pwd'
                    sh 'pwd && ls -la'
                    // Set executable permissions for the Maven Wrapper
                    sh 'chmod +x mvnw'
                    // Build the Quarkus project using Maven Wrapper
                    sh './mvnw package -Dquarkus.package.jar.type=uber-jar'
                    
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
