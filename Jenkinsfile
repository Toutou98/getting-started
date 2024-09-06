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
        stage('Debug Info') {
            steps {
                echo "Pod created successfully."
                echo "Checking if checkout was successful."
            }
        }
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
                    sh 'mvn package -Dquarkus.package.jar.type=uber-jar'
                    sh 'pwd'
                    sh 'ls -la'
                    sh 'ls -la target/'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                container('docker') {
                    script {
                        // Build the Docker image using your specific Dockerfile (Dockerfile.jvm)
                        sh 'docker build -f src/main/docker/Dockerfile.toutou -t getting-started:1.0 .'
                    }
                }
            }
        }
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
        cleanWs()
    }
}

}
