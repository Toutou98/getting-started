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
                - --insecure-registry=nexus-docker.nexus.svc.cluster.local:8083
                securityContext:
                  privileged: true
              - name: helm
                image: alpine/helm:3.15.4
                command:
                - cat
                tty: true
            """
        }
    }
    environment {
        HELM_URL = "http://my-nexus-nexus-repository-manager.nexus:8081/repository/helm-local-repo/"
        DOCKER_REGISTRY = "http://nexus-docker.nexus.svc.cluster.local:8083"
        DOCKER_REGISTRY_DOMAIN = "nexus-docker.nexus.svc.cluster.local:8083"
        DOCKER_IMAGE = "getting-started:1.0.0"
    }
    stages {
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn package -Dquarkus.package.jar.type=uber-jar'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                dockerSteps.buildDockerImage('src/main/docker/Dockerfile.toutou', env.DOCKER_IMAGE, env.DOCKER_REGISTRY_DOMAIN)
            }
        }
        stage('Package Helm Chart') {
            steps {
                helmSteps.packageChart('quarkus-app', env.HELM_URL)
            }
        }
        stage('Deploy Helm Chart') {
            steps {
                helmSteps.deployChart('quarkus-app', env.HELM_URL)
            }
        }
        // stage('Build') {
        //     steps {
        //         container('maven') {
        //             sh 'mvn package -Dquarkus.package.jar.type=uber-jar'
        //         }
        //     }
        // }
        // stage('Build Docker Image') {
        //     steps {
        //         container('docker') {
        //             script {
        //                 // Build the Docker image using your specific Dockerfile (Dockerfile.jvm)
        //                 sh "docker build -f src/main/docker/Dockerfile.toutou -t ${DOCKER_IMAGE} ."
                        
        //                 // Authenticate with Nexus Docker registry
        //                 withCredentials([usernamePassword(credentialsId: 'nexus', usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')]) {
        //                     sh 'pwd'
        //                     sh "echo $NEXUS_PASSWORD | docker login ${DOCKER_REGISTRY} -u $NEXUS_USERNAME --password-stdin"
        //                     sh "docker tag ${DOCKER_IMAGE} ${DOCKER_REGISTRY_DOMAIN}/${DOCKER_IMAGE}"
        //                     sh "docker push ${DOCKER_REGISTRY_DOMAIN}/${DOCKER_IMAGE}"
        //                 }
        //             }
        //         }
        //     }
        // }
        // stage('Package Helm Chart') {
        //     steps {
        //         container('helm') {
        //             script {
        //                 // Package the Helm chart from the existing directory
        //                 sh 'helm package quarkus-app'
        //                 // Push the Helm chart to Nexus using curl
        //                 withCredentials([usernamePassword(credentialsId: 'nexus', usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')]) 
        //                 {
        //                     sh 'curl -u $NEXUS_USERNAME:$NEXUS_PASSWORD --upload-file quarkus-app-*.tgz $HELM_URL'
        //                 }
        //             }
        //         }
        //     }
        // }
        // stage('Deploy Helm Chart') {
        //     steps {
        //         container('helm') {
        //             script {
        //                 sh "helm repo add helm-local ${HELM_URL}"
        //                 sh 'helm repo update'
        //                 def releaseExists = sh(script: "helm list -q | grep -w 'quarkus-app'", returnStatus: true) == 0
                
        //                 if (releaseExists) {
        //                     // Upgrade the release if it already exists
        //                     echo 'Release already exists. Upgrading...'
        //                     sh 'helm upgrade quarkus-app helm-local/quarkus-app'
        //                 } else {
        //                     // Install the release if it does not exist
        //                     echo 'Release does not exist. Installing...'
        //                     sh 'helm install quarkus-app helm-local/quarkus-app'
        //                 }
        //             }
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