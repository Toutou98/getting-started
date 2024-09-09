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
                securityContext:
                  privileged: true
                command:
                - dockerd
                - --host=unix:///var/run/docker.sock
                - --host=tcp://0.0.0.0:2375
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
