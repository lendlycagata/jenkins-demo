pipeline {
  agent any

  environment {
    PROJECT_ID = 'off-net-dev'
    REGION = 'northamerica-northeast1'
    CLUSTER = 'jenkins-demo'
    IMAGE = 'jenkins-demo'
    REPO = "northamerica-northeast1-docker.pkg.dev/${PROJECT_ID}/jenkins-demo/${IMAGE}"
    GOOGLE_APPLICATION_CREDENTIALS = credentials('gcp-service-account')
  }

  stages {

    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/lendlycagata/jenkins-demo.git'
      }
    }

    stage('Build JAR') {
      steps {
        sh './mvnw clean package -DskipTests'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $REPO:$BUILD_NUMBER .'
      }
    }

    stage('Push to Artifact Registry') {
      steps {
        withCredentials([file(credentialsId: 'gcp-service-account', variable: 'GCP_KEY')]) {
          sh '''
            gcloud auth activate-service-account --key-file=$GCP_KEY
            gcloud auth configure-docker ${REGION}-docker.pkg.dev --quiet
            docker push $REPO:$BUILD_NUMBER
          '''
        }
      }
    }

    stage('Deploy to GKE') {
      steps {
        withCredentials([file(credentialsId: 'gcp-service-account', variable: 'GCP_KEY')]) {
          sh '''
            gcloud auth activate-service-account --key-file=$GCP_KEY
            gcloud container clusters get-credentials $CLUSTER --region=$REGION --project=$PROJECT_ID
            kubectl set image deployment/jenkins-demo jenkins-demo=$REPO:$BUILD_NUMBER
          '''
        }
      }
    }
  }

  post {
    success {
      echo "✅ Deployed successfully to GKE!"
    }
    failure {
      echo "❌ Build or Deploy failed!"
    }
  }
}
