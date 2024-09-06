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
                - dockerd --host=unix:///var/run/docker.sock --host=tcp://0.0.0.0:2375 --insecure-registry host.docker.internal:8081
                env:
                - name: DOCKER_TLS_CERTDIR
                value: ""
                - name: DOCKER_OPTS
                value: "--insecure-registry host.docker.internal:8081"
                tty: true
                securityContext:
                    privileged: true
                volumeMounts:
                - name: docker-socket
                  mountPath: /var/run/docker.sock
              - name: helm
                image: alpine/helm:3.12.0
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
        NEXUS_URL = "http://host.docker.internal:8081/repository/helm-local/"
        NEXUS_REPO = "helm-local"
        DOCKER_REGISTRY = "http://host.docker.internal:8081/repository/docker-local/"
        DOCKER_IMAGE = "getting-started:1.0.0"
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
                        sh "docker build -f src/main/docker/Dockerfile.toutou -t ${DOCKER_IMAGE} ."
                        // Authenticate with Nexus Docker registry
                        withEnv(["DOCKER_OPTS=--insecure-registry host.docker.internal:8081"]) {
                            withCredentials([usernamePassword(credentialsId: 'nexus', usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')]) {
                                sh 'pwd'
                                sh "echo $NEXUS_PASSWORD | docker login ${DOCKER_REGISTRY} -u $NEXUS_USERNAME --password-stdin"
                                sh "docker tag ${DOCKER_IMAGE} ${DOCKER_REGISTRY}${DOCKER_IMAGE}"
                                sh "docker push ${DOCKER_REGISTRY}${DOCKER_IMAGE}"
                            }
                        
                        }
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
                        // Push the Helm chart to Nexus using curl
                        withCredentials([usernamePassword(credentialsId: 'nexus', usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')]) 
                        {
                            sh 'curl -u $NEXUS_USERNAME:$NEXUS_PASSWORD --upload-file quarkus-app-*.tgz $NEXUS_URL'
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
