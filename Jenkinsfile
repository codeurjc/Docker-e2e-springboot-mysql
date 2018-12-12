worker = "worker-${UUID.randomUUID().toString()}"

podTemplate(label: worker, containers: [
  containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true, envVars: [
    envVar(key: 'TAG', value: "${TAG}")
    ])
  ],
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
  ]) {

  node(worker) {

    stage('Cloning the repo...') {
      git 'https://github.com/codeurjc/Docker-e2e-springboot-mysql'
    }

    stage('Building the image...') {
      container('docker') {
        sh """
          docker build -t codeurjc/springboot-e2e:\${TAG} .
        """
      }
    }

    stage('Cleaning dangling images...') {
      container('docker') {
        sh 'docker images --quiet --filter=dangling=true | xargs --no-run-if-empty docker rmi -f'
      }
    }

    stage('Pushing to registry...') {
      container('docker') {
        withCredentials([[$class: 'UsernamePasswordMultiBinding',
          credentialsId: '36779dc9-b8ed-4d79-beda-2e624552b73f',
          usernameVariable: 'DOCKER_HUB_USER',
          passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
            sh """
              docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASSWORD}
              docker push codeurjc/springboot-e2e:\${TAG}
            """
        }
      }
    }
  }
}
