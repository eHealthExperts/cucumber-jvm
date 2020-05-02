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
        withDocker(from:'maven:3-jdk-11'){
            script{
                mvn "clean verify -Psonar"
            }
        }
      }
    }
    stage("Publish (optional)") {
      steps {
          withDocker(from:'maven:3-jdk-11'){
            script{
                mvn "-DskipTests=true -Pehex-deploy -V -U -B deploy"
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
