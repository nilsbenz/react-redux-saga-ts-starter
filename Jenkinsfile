#!groovy

//
// Notification settings.
//
def SLACK_DOMAIN = "adcubum"
def SLACK_CHANNEL = "#nbenz-build"
def SLACK_TOKEN = "z4MN0likx2G9V2jbPmiJT8ZD"
def BUILD_INFO = "<${JOB_URL}|${JOB_NAME}> <${BUILD_URL}|#${BUILD_NUMBER}>" as String

final String agentProjectLabel = 'adcubum-application'
final String agentDockerLabel = 'docker'
final String jenkinsContainerCustomWorkspace = '/home/jenkins/workspaces/build'

pipeline {
  agent {
    label "${agentProjectLabel} && ${agentDockerLabel}"
  }

  triggers { pollSCM("H/2 * * * 1-5") }

  options { buildDiscarder(logRotator(numToKeepStr: "10", artifactNumToKeepStr: "10")) }

  environment {
    // CI is used in scripts to determine whether or not the script is executed on CI system.
    CI = 'true'
    HOME = '.'
  }


  stages {
    stage('prepare') {
      agent {
        docker {
          image "node:10"
          reuseNode true
          customWorkspace jenkinsContainerCustomWorkspace
        }
      }
      steps{
        step([$class: "StashNotifier"])
        sh 'npm install --quiet'
      }
    }
    stage('lint') {
      agent {
        docker {
          image "node:10"
          reuseNode true
          customWorkspace jenkinsContainerCustomWorkspace
        }
      }
      steps{
        sh 'npm run lint'
      }
    }
    stage('test') {
      agent {
        docker {
          image "node:10"
          reuseNode true
          customWorkspace jenkinsContainerCustomWorkspace
        }
      }
      steps{
        sh 'npm run test'
      }
    }
    stage('bundle') {
      agent {
        docker {
          image "node:10"
          reuseNode true
          customWorkspace jenkinsContainerCustomWorkspace
        }
      }
      steps{
        sh 'npm run build'
      }
    }
    stage('docker') {
      steps {
        script {
          docker.withRegistry('https://internal-tools.adcr.io', 'artifactory-docker-erp-internal-tools-publisher') {
            def image = docker.build("internal-tools.adcr.io/syrius-itsupport-grpc-testing-webapp:${GIT_BRANCH.replace('/', '-')}-${BUILD_NUMBER}")
            image.push()
            if(GIT_BRANCH == 'develop') {
              def devImage = docker.build("internal-tools.adcr.io/syrius-itsupport-grpc-testing-webapp:develop-latest")
              devImage.push()
            } else if(GIT_BRANCH == 'master') {
              def prodImage = docker.build("internal-tools.adcr.io/syrius-itsupport-grpc-testing-webapp:latest")
              prodImage.push()
            }
          }
        }
      }
    }
  }

  post {
    regression {
      slackSend teamDomain: SLACK_DOMAIN, token: SLACK_TOKEN, channel: SLACK_CHANNEL, color: "danger", message: "Build broken: ${BUILD_INFO}"
    }
    fixed {
      slackSend teamDomain: SLACK_DOMAIN, token: SLACK_TOKEN, channel: SLACK_CHANNEL, color: "good", message: "Build fixed: ${BUILD_INFO}"
    }
    always {
      // Stash must be notified about the build status in order to display
      // the build result.
      script { currentBuild.result = currentBuild.result ?: "SUCCESS" }
      step([$class: "StashNotifier"])
    }
    cleanup {
      cleanWs()
    }
  }

}
