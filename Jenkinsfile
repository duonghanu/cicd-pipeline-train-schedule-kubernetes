pipeline {
  agent any
  environment {
    // be sure to replace "duonghanu" with your own Docker Hub username
    DOCKER_IMAGE_NAME = "duonghanu/train-schedule"
  }

  stages {
    stage('Build') {
      steps {
        echo 'Running build automation'
        sh './gradlew build --no-daemon'
        archiveArtifacts artifacts: 'dist/trainSchedule.zip'
      }
    } // end Build stage

    stage('Build Docker Image') {
      when {
        branch 'master'
      }

      steps {
        script {
          app = docker.build(DOCKER_IMAGE_NAME)
          app.inside {
            sh 'echo Hello, World!'
          }
        }
      }
    } // end Build Docker Image stage

    stage('Push Docker Image') {
      when {
        branch 'master'
      }

      steps {
        script {
          docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
            app.push("${env.BUILD_NUMBER}")
            app.push('latest')
          }
        }
      }
    } // end Push Docker Image stage

    stage('DeployToProduction') {
      when {
        branch 'master'
      }

      steps {
        input 'Deploy to Production?'
        milestone(1)
        kubernetesDeploy(
          kubeconfigId: 'kubeconfig',
          configs: 'train-schedule-kube.yml',
          enableConfigSubstitution: true
        )
      }
    } // end Deploy To Production stage
  } // end stages
} // end pipeline
