@Library("jenkins-shared@main") _ 

script {
    myPipeline.call(app_name: 'quarkus-app'
                    dockerfile: 'src/main/docker/Dockerfile.toutou'
                    helm_url: 'http://my-nexus-nexus-repository-manager.nexus:8081/repository/helm-local-repo/',
                    docker_registry: 'http://nexus-docker.nexus.svc.cluster.local:8083',
                    docker_registry_domain: 'nexus-docker.nexus.svc.cluster.local:8083',
                    docker_image: 'getting-started:1.0.0')
}