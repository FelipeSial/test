def artifactory_url="https://artifactory.build.ge.com"


pipeline {

  agent { label "Sdp-Container-agent" }

  parameters {
     string(name: 'version', defaultValue: '3.2.beta', description: 'version to save (only for master branch)')
  }

  environment {
    PATH = "${workspace}:$PATH"
    ARTIFACTORY_API_KEY = credentials('SDP_ARTIFACTORY_API_KEY')
  }

  stages {

    stage ('Create file version.info') {

      steps {
        print "branch :  ${env.GIT_BRANCH}"
        print "commit :  ${env.GIT_COMMIT}"


        sh """
           if [ ${env.GIT_BRANCH} = "detached" ]; then
              echo  ${env.GIT_COMMIT}  > /tmp/BRANCH.txt
              echo branch: `cat /tmp/BRANCH.txt`   --- commit: ${env.GIT_COMMIT} > version.info
           else
              echo  ${env.GIT_BRANCH} | sed 's/.*\\///' > /tmp/BRANCH.txt
              echo branch: `cat /tmp/BRANCH.txt`   --- commit: ${env.GIT_COMMIT} > version.info
          fi;
        """

      }

    }

    stage ('build viper_lib tar'){
      steps {
        sh """
          tar --exclude='.git/*'  -zcvf /tmp/viper.tgz .
        """
      }
    }

    stage('Save viper_lib to artifactory') {
      parallel {
        stage ('master') {
          when { branch 'master' }
          steps {
              sh """
                  set +x
                  curl -H 'X-JFrog-Art-Api: ${env.ARTIFACTORY_API_KEY}' ${artifactory_url}/PAQLY/ge.power.sws.sdp/viper_lib/viper_lib-${params.version}.tgz -T /tmp/viper.tgz
    			        curl -H 'X-JFrog-Art-Api: ${env.ARTIFACTORY_API_KEY}' -X PUT ${artifactory_url}/api/storage/PAQLY/ge.power.sws.sdp/viper_lib/viper_lib-${params.version}.tgz?properties=version=branch:`cat /tmp/BRANCH.txt`---commit:${env.GIT_COMMIT}
                  curl -H 'X-JFrog-Art-Api: ${env.ARTIFACTORY_API_KEY}' ${artifactory_url}/PAQLY/ge.power.sws.sdp/viper_lib/viper_lib-latest.tgz -T /tmp/viper.tgz
                  curl -H 'X-JFrog-Art-Api: ${env.ARTIFACTORY_API_KEY}' -X PUT ${artifactory_url}/api/storage/PAQLY/ge.power.sws.sdp/viper_lib/viper_lib-latest.tgz?properties=version=branch:`cat /tmp/BRANCH.txt`---commit:${env.GIT_COMMIT}
               """
          }
        }
        stage ('not master') {
          when { not { branch 'master' } }
          steps {
              sh """
                  set +x
                  curl -H 'X-JFrog-Art-Api: ${env.ARTIFACTORY_API_KEY}' ${artifactory_url}/PAQLY/ge.power.sws.sdp/viper_lib/viper_lib-`cat /tmp/BRANCH.txt`.tgz -T /tmp/viper.tgz
    			        curl -H 'X-JFrog-Art-Api: ${env.ARTIFACTORY_API_KEY}' -X PUT ${artifactory_url}/api/storage/PAQLY/ge.power.sws.sdp/viper_lib/viper_lib-`cat /tmp/BRANCH.txt`.tgz?properties=version=branch:`cat /tmp/BRANCH.txt`---commit:${env.GIT_COMMIT}
               """
          }
        }
      }
    }


  }
}
