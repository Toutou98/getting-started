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
                image: docker:24.0.2-dind
                command:
                - dockerd
                - --host=unix:///var/run/docker.sock
                - --host=tcp://0.0.0.0:2375
                - --insecure-registry=host.docker.internal:8081
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
        NEXUS_URL = "http://host.docker.internal:8082/repository/helm-local-repo/"
        NEXUS_REPO = "helm-local"
        DOCKER_REGISTRY = "http://host.docker.internal:8082/repository/docker-local-registry/"
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

    }
    post {
        always {
            cleanWs()
        }
    }
}
