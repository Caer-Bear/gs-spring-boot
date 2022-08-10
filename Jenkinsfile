node {
  stage('SCM') {
    checkout scm
  }
  stage('SonarQube Analysis') {
    withSonarQubeEnv() {
      dir("complete") {
        sh "./gradlew sonarqube"
      }

    }
  }
}