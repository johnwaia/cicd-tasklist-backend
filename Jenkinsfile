node {
  def nodejs = tool name: 'Node20', type: 'jenkins.plugins.nodejs.tools.NodeJSInstallation'
  env.PATH = "${nodejs}/bin:${env.PATH}"

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
}