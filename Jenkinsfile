library 'pipeline-sugar'

pipeline {

  agent any

  options {
    buildDiscarder(logRotator(numToKeepStr: "5"))
    skipDefaultCheckout()
    timeout(time: 30, unit: "MINUTES")
    disableConcurrentBuilds()
    timestamps()
  }

  environment {
    DEPLOY_BRANCH = "master"
    RECIPIENTS = "connector-devs@ehealthexperts.de"
    NEXUS_CREDENTIALS = credentials("a24c724c-4a4e-4d9b-bb61-5ea3d98a4c6f")
  }

  stages {
    stage("Checkout SCM") {
      steps {
        checkoutScm()
      }
    }

    stage("Build & Test") {
      steps {
        withDocker(){
            script{
                maven goal: "clean verify -Psonar"
            }
        }
      }
    }
    stage("Publish (optional)") {
      steps {
          withDocker(){
            script{
                maven goal: "deploy", param: "-DaltReleaseDeploymentRepository=ekon-maven-releases::https://artifacts.ehex.de/repository/ekon-maven-releases -DaltSnapshotDeploymentRepositry=ekon-maven-releases::https://artifacts.ehex.de/repository/ehex-maven-releases -DskipTests=true -Pehex-deploy  -Ddocker.username=${NEXUS_CREDENTIALS_USR} -Ddocker.password=${NEXUS_CREDENTIALS_PSW}"
            }
          }
      }
    }
  }
  post {
          success {
              archive '**/target/*.jar'
          }
          failure {
              sendEmail cond: "fail"
          }
          fixed {
              sendEmail cond: "fixed"
          }
          cleanup {
              cleanWs()
          }
      }

}
