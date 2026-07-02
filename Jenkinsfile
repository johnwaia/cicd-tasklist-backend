node {
  def nodejs = tool name: 'Node20', type: 'jenkins.plugins.nodejs.tools.NodeJSInstallation'
  env.PATH = "${nodejs}/bin:${env.PATH}"

  def dockerImage = "johnwaia/cicd-tasklist-backend"

  stage('SCM') {
    checkout scm
  }
  stage('Install & Test') {
    sh 'npm ci'
    sh 'npm run test:coverage'
  }
  stage('SonarQube Analysis') {
    def scannerHome = tool 'SonarScanner';
    withSonarQubeEnv() {
      sh "${scannerHome}/bin/sonar-scanner"
    }
  }
  stage('Docker Build') {
    sh "docker build -t ${dockerImage}:${env.BUILD_NUMBER} -t ${dockerImage}:latest ."
  }
  stage('Docker Push') {
    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
      sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
      sh "docker push ${dockerImage}:${env.BUILD_NUMBER}"
      sh "docker push ${dockerImage}:latest"
    }
  }
}