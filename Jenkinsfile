pipeline {
    agent {
        kubernetes {
            yaml """
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: docker
                image: docker:24.0.2
                command:
                - cat
                tty: true
                volumeMounts:
                - name: docker-socket
                  mountPath: /var/run/docker.sock
              volumes:
              - name: docker-socket
                hostPath:
                  path: /var/run/docker.sock
            """
        }
    }
    stages {
        stage('Clone') {
            steps {
                container('docker') {
                    git branch: 'main', url: 'https://github.com/Toutou98/getting-started.git'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                container('docker') {
                    script {
                        // Build the Docker image using your specific Dockerfile (Dockerfile.jvm)
                        sh 'docker build -f src/main/docker/Dockerfile.jvm -t getting-started:1.0 .'
                    }
                }
            }
        }
        // stage('Build') {
        //     steps {
        //         container('maven') {
        //             sh 'mvn package -Dquarkus.package.jar.type=uber-jar'
        //         }
        //     }
        // }
        // stage('Archive') {
        //     steps {
        //         container('maven') {
        //             archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
           
        //         }
        //     }
        // }
    }
    post {
        always {
            // Clean up workspace
            cleanWs()
        }
    }
}
