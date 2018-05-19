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
          withDockerContainer(image: 'dvitali/pixelc-build-container:6', args: '--privileged') {
            sh 'bash -c "mkdir -p out/ubuntu/rootfs"'
            sh 'UID=0 GID=0 DISTRO=ubuntu SYSROOT=$(pwd)/out/$DISTRO/rootfs TOP=$(pwd) ./build.sh'
          }
        }
      }
    }

    stage('Publish') {
      steps {
        archiveArtifacts 'rootfs-builder/out/*_rootfs.tar.gz'
        withDockerContainer(image: 'dvitali/pixelc-build-container:6'){
          echo "Deleting release from github before creating new one"
          VERSION_NAME = sh(returnStdout: true, script: 'cat VERSION_NAME').trim()
          sh "github-release delete --user ${GITHUB_ORGANIZATION} --repo ${GITHUB_REPO} --tag ${VERSION_NAME}"

          echo "Creating a new release in github"
          sh "github-release release --user ${GITHUB_ORGANIZATION} --repo ${GITHUB_REPO} --tag ${VERSION_NAME} --name ${VERSION_NAME}"

          echo "Uploading the artifacts into github"
          sh "github-release upload --user ${GITHUB_ORGANIZATION} --repo ${GITHUB_REPO} --tag ${VERSION_NAME} --name ${DISTRO}-${VERSION_NAME}_rootfs.tar.gz --file out/ubuntu_rootfs.tar.gz"
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
