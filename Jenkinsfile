pipeline {
  agent any

  environment {
    REPO = "https://github.com/simran-kanase/simran-portfolio.git"
    GH_BRANCH = "gh-pages"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout([$class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[url: env.REPO, credentialsId: 'github-creds']]
        ])
      }
    }

    stage('Prepare site') {
      steps {
        sh 'mkdir -p build && cp -r index.html about.html css build/ || true'
      }
    }

    stage('Publish to gh-pages') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
          sh '''
            set -e
            rm -rf gh-temp
            git clone --branch ${GH_BRANCH} ${REPO} gh-temp || \
              (git clone ${REPO} gh-temp && cd gh-temp && git checkout --orphan ${GH_BRANCH})

            # delete old site files
            rm -rf gh-temp/*

            # copy new site files
            cp -r build/* gh-temp/

            cd gh-temp
            git config user.email "jenkins@local"
            git config user.name "${GIT_USER}"
            git add --all
            if git diff --cached --quiet; then
              echo "No changes to deploy"
            else
              git commit -m "Automated update from Jenkins"
              git push https://${GIT_USER}:${GIT_TOKEN}@github.com/simran-kanase/simran-portfolio.git ${GH_BRANCH}
            fi
          '''
        }
      }
    }
  }
}
