pipeline {
    agent {
        kubernetes {
            yaml """
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: maven
                image: maven:3.9.9-ibm-semeru-21-jammy
                command:
                - cat
                tty: true
            """
        }
    }
    stages {
        stage('Clone') {
            steps {
                container('maven') {
                    git branch: 'main', url: 'https://github.com/Toutou98/getting-started.git'
                }
            }
        }
        stage('Build') {
            steps {
                container('maven') {
                    catchError {
                        sh 'mvn package -Dquarkus.package.jar.type=uber-jar'
                    }                    
                }
            }
        }
        stage('Archive') {
            steps {
                container('maven') {
                    archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
           
                }
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
