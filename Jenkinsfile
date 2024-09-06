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
              - name: helm
                image: alpine/helm:3.12.0
                command:
                - cat
                tty: true
              - name: curl
                image: curlimages/curl:7.85.0
                command:
                - cat
                tty: true
              volumes:
              - name: docker-socket
                hostPath:
                  path: /var/run/docker.sock
            """
        }
    }
    environment {
        NEXUS_URL = "http://localhost:8081/repository/helm-local/"
        NEXUS_REPO = "helm-local"
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
        stage('Package Helm Chart') {
            steps {
                container('helm') {
                    script {
                        // Package the Helm chart from the existing directory
                        sh 'helm package quarkus-app'
                    }
                }
            }
        }
        stage('Push Helm Chart to Nexus') {
            steps {
                container('curl') {
                    script {
                        // Push the Helm chart to Nexus using curl
                        withCredentials([usernamePassword(credentialsId: 'nexus', usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')]) 
                        {
                            echo "NEXUS_USERNAME: ${NEXUS_USERNAME}"
                            sh 'curl -u $NEXUS_USERNAME:$NEXUS_PASSWORD --upload-file quarkus-app-*.tgz $NEXUS_URL'
                        }
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
