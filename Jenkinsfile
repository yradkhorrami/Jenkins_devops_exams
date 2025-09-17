pipeline {
  agent any
  options { timestamps() }

  environment {
    // Use Docker Hub (matches what you pushed): change if you use GitLab registry
    MOVIE_REPO = 'yradkhorrami/movie-service'
    CAST_REPO  = 'yradkhorrami/cast-service'
    IMAGE_TAG  = "${env.BUILD_NUMBER}"        // or: sh(returnStdout:true, script:'git rev-parse --short HEAD').trim()
  }

  stages {
    stage('Docker Build: cast-service') {
      steps {
        sh """
          docker build -t ${CAST_REPO}:${IMAGE_TAG} cast-service
        """
      }
    }

    stage('Docker Push (both)') {
      environment { DOCKER_PASS = credentials('DOCKER_HUB_PASS') } // id with user/pass or token
      steps {
        sh '''
        #!/bin/bash
        set -eux
        echo "$DOCKER_PASS" | docker login -u "$DOCKER_ID" --password-stdin
        docker push ${MOVIE_REPO}:${IMAGE_TAG}
        docker push ${CAST_REPO}:${IMAGE_TAG}
        '''
      }
    }

    // ---- Minimal CD bits below ----
    stage('Deploy to Dev (Helm)') {
      steps {
        sh """
          # movie
          helm upgrade --install movie charts -n dev \
            -f charts/env/dev.yaml \
            --set image.repository=${MOVIE_REPO} \
            --set image.tag=${IMAGE_TAG}

          # cast
          helm upgrade --install cast charts -n dev \
            -f charts/env/cast-dev.yaml \
            --set image.repository=${CAST_REPO} \
            --set image.tag=${IMAGE_TAG}

          # wait for rollouts (fail fast if unhealthy)
          kubectl -n dev rollout status deploy/movie-fastapiapp --timeout=180s
          kubectl -n dev rollout status deploy/cast-fastapiapp  --timeout=180s
        """
      }
    }

    stage('Smoke Test (Dev)') {
      steps {
        sh """
          # movies swagger should load
          curl -fsS http://127.0.0.1:30007/api/v1/movies/openapi.json >/dev/null

          # casts swagger should load
          curl -fsS http://127.0.0.1:30008/api/v1/casts/openapi.json  >/dev/null
        """
      }
    }

    // Optional: manual promotion & deploy to QA/Staging
    stage('Promote to QA?') {
      when { branch 'main' }
      steps { input message: 'Deploy to QA?' }
    }

    stage('Deploy to QA (optional)') {
      when{
		beforeAgent true
		branch 'main'
	  }
      steps {
        sh """
          helm upgrade --install movie charts -n qa  -f charts/env/qa.yaml \
            --set image.repository=${MOVIE_REPO} --set image.tag=${IMAGE_TAG}
          helm upgrade --install cast  charts -n qa  -f charts/env/cast-qa.yaml \
            --set image.repository=${CAST_REPO}  --set image.tag=${IMAGE_TAG}
          kubectl -n qa rollout status deploy/movie-fastapiapp --timeout=180s
          kubectl -n qa rollout status deploy/cast-fastapiapp  --timeout=180s
        """
      }
    }
  }

  post {
    always {
      // Optional cleanup
      sh 'docker logout || true'
    }
  }
}
