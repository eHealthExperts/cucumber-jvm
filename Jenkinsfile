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
                maven goal: "deploy", param: "-DaltReleaseDeploymentRepository=ehex-maven-releases::https://artifacts.ehex.de/repository/ehex-maven-releases -DaltSnapshotDeploymentRepository=ehex-maven-snapshots::https://artifacts.ehex.de/repository/ehex-maven-snapshots -DskipTests=true -Pehex-deploy"
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
