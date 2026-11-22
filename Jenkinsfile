pipeline {
  agent any

  options {
    timestamps()
    timeout(time: 20, unit: 'MINUTES')
  }

  environment {
    PATH = "${env.HOME}/.local/bin:${env.PATH}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Preflight: tooling check') {
      steps {
        sh '''
          if ! command -v docker >/dev/null 2>&1; then
            echo "ERROR: docker is not installed on this agent. Install Docker Engine/Desktop on the Jenkins agent before running this pipeline."
            exit 1
          fi
          docker version
          docker compose version

          if ! command -v postman >/dev/null 2>&1; then
            echo "Postman CLI not found, installing..."
            curl -o- "https://dl-cli.pstmn.io/install/unix.sh" | sh
          fi
          postman --version
        '''
      }
    }

    stage('Provide .env.ci') {
      steps {
        withCredentials([
          string(credentialsId: 'ci-postgres-user',     variable: 'POSTGRES_USER'),
          string(credentialsId: 'ci-postgres-password', variable: 'POSTGRES_PASSWORD'),
          string(credentialsId: 'ci-postgres-db',        variable: 'POSTGRES_DB')
        ]) {
          sh '''
            cat > .env.ci <<EOF
POSTGRES_USER=${POSTGRES_USER}
POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
POSTGRES_DB=${POSTGRES_DB}
DB_HOST=db
DB_PORT=5432
API_PORT=3000
EOF
          '''
        }
      }
    }

    stage('Build images') {
      steps {
        sh 'docker compose --env-file .env.ci build'
      }
    }

    stage('Start stack') {
      steps {
        sh 'docker compose --env-file .env.ci up -d'
      }
    }

    stage('Wait for API readiness') {
      steps {
        sh '''
          for i in $(seq 1 30); do
            if curl -fs http://localhost:3000/health; then
              echo "API healthy"
              exit 0
            fi
            echo "Waiting for API... ($i/30)"
            sleep 2
          done
          echo "API did not become healthy in time"
          docker compose --env-file .env.ci logs
          exit 1
        '''
      }
    }

    stage('Postman API tests') {
      steps {
        sh '''
          postman collection run "postman/collections/Tasks API" \
            -e "postman/environments/CI.environment.yaml" \
            --reporter cli \
            --output reports/postman
        '''
      }
    }
  }

  post {
    always {
      sh 'docker compose --env-file .env.ci down -v || true'
      archiveArtifacts artifacts: 'reports/**', allowEmptyArchive: true
      sh 'rm -f .env.ci'
    }
  }
}
