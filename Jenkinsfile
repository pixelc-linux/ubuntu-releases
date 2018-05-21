pipeline {
 agent any
 options {
    ansiColor('xterm')
 }

  environment {
      GITHUB_TOKEN = credentials('github-token-pixelc-linux')
      GITHUB_ORGANIZATION = "pixelc-linux"
      GITHUB_REPO = "ubuntu-releases"
      DISTRO = "Ubuntu"
  }

 stages {
    stage('Checkout') {
      steps {
        sh 'git clone https://github.com/pixelc-linux/rootfs-builder rootfs-builder'
      }
    }
    
    stage('Build') {
      steps {
        dir('rootfs-builder') {
          withDockerContainer(image: 'dvitali/pixelc-build-container:7', args: '--privileged') {
            sh 'bash -c "mkdir -p out/ubuntu/rootfs"'
            sh 'UID=0 GID=0 DISTRO=ubuntu SYSROOT=$(pwd)/out/$DISTRO/rootfs TOP=$(pwd) ./build.sh'
          }
        }
      }
    }

    stage('Publish') {
      steps {
        script {
           env.VERSION_NAME = sh(returnStdout: true, script: 'cat VERSION_NAME').trim()
        }
        archiveArtifacts 'rootfs-builder/out/*_rootfs.tar.gz'
        withDockerContainer(image: 'dvitali/pixelc-build-container:7'){
          sh ". /etc/environment"
          sh "echo $PATH"
          echo "Deleting release from github before creating new one"
          sh "/opt/go/bin/github-release delete --user ${GITHUB_ORGANIZATION} --repo ${GITHUB_REPO} --tag ${env.VERSION_NAME}"

          echo "Creating a new release in github"
          sh "/opt/go/bin/github-release release --user ${GITHUB_ORGANIZATION} --repo ${GITHUB_REPO} --tag ${env.VERSION_NAME} --name ${VERSION_NAME}"

          echo "Uploading the artifacts into github"
          sh "/opt/go/bin/github-release upload --user ${GITHUB_ORGANIZATION} --repo ${GITHUB_REPO} --tag ${env.VERSION_NAME} --name ${DISTRO}-${VERSION_NAME}_rootfs.tar.gz --file rootfs-builder/out/ubuntu_rootfs.tar.gz"
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
