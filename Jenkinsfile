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
    stage("Sonar Report & Quality Gate") {
      when {
        branch DEPLOY_BRANCH
      }
      steps {
        withDocker() {
          script {
            steps.withSonarQubeEnv("sonar") {
              maven param: 'sonar:sonar -Dsonar.scm.disabled=true -Dsonar.sources=src/main -Dsonar.projectName=commons'
            }
          }
        }
        timeout(time: 1, unit: 'HOURS') {
          waitForQualityGate abortPipeline: false
        }
      }
    }
    
    stage("Publish (optional)") {
      when {
        allOf {
          branch DEPLOY_BRANCH
        }
      }
      steps {
          withDocker(){
            script{
                maven goal: "deploy", param: "-DskipTests=true"
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
