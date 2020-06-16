def artifactory_url="https://artifactory.build.ge.com/"
pipeline {
    agent any
     environment {
    PATH = "${workspace}:$PATH"
    ARTIFACTORY_API_KEY = credentials('devcloud')
  }
    stages {
        stage('Build') { 
            steps {
               teps {
              sh """
                  set +x
                  curl -H 'X-JFrog-Art-Api: ${env.ARTIFACTORY_API_KEY}' ${artifactory_url}/PAQLY/Install_kits/emp33windows/eterra/e-terracomm.tgz -T /tmp/viper.tgz
            }
        }
        
    }
}
