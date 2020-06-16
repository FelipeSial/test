def artifactory_url="https://artifactory.build.ge.com/"


pipeline {

  agent { label "dind" }

  parameters {
     string(name: 'version', defaultValue: '3.2.beta', description: 'version to save (only for master branch)')
  }

  environment {
    PATH = "${workspace}:$PATH"
    ARTIFACTORY_API_KEY = credentials('devcloud')
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
          apt-get update -y
          apt-get install zip -y
          tar --exclude='.git/*'  -zcvf /tmp/viper.tgz .
        """
      }
    }

    stage ('build functions zip'){
      steps {
        sh """
          zip -r /tmp/viperlib_functions.zip viper-properties.txt viper-deploy.ps1 core/functions core/scripts
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
                  
                  curl -H 'X-JFrog-Art-Api: ${env.ARTIFACTORY_API_KEY}' ${artifactory_url}/PAQLY/ge.power.sws.sdp/viper_lib/viperlib_functions-${params.version}.zip -T /tmp/viperlib_functions.zip
    			        curl -H 'X-JFrog-Art-Api: ${env.ARTIFACTORY_API_KEY}' -X PUT ${artifactory_url}/api/storage/PAQLY/ge.power.sws.sdp/viper_lib/vviperlib_functions-${params.version}.zip?properties=version=branch:`cat /tmp/BRANCH.txt`---commit:${env.GIT_COMMIT}
                  curl -H 'X-JFrog-Art-Api: ${env.ARTIFACTORY_API_KEY}' ${artifactory_url}/PAQLY/ge.power.sws.sdp/viper_lib/viperlib_functions-latest.zip -T /tmp/viperlib_functions.zip
                  curl -H 'X-JFrog-Art-Api: ${env.ARTIFACTORY_API_KEY}' -X PUT ${artifactory_url}/api/storage/PAQLY/ge.power.sws.sdp/viper_lib/viperlib_functions-latest.zip?properties=version=branch:`cat /tmp/BRANCH.txt`---commit:${env.GIT_COMMIT}
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
 
                  curl -H 'X-JFrog-Art-Api: ${env.ARTIFACTORY_API_KEY}' ${artifactory_url}/PAQLY/ge.power.sws.sdp/viper_lib/viperlib_functions-`cat /tmp/BRANCH.txt`.zip -T /tmp/viperlib_functions.zip
    			        curl -H 'X-JFrog-Art-Api: ${env.ARTIFACTORY_API_KEY}' -X PUT ${artifactory_url}/api/storage/PAQLY/ge.power.sws.sdp/viper_lib/viperlib_functions-`cat /tmp/BRANCH.txt`.zip?properties=version=branch:`cat /tmp/BRANCH.txt`---commit:${env.GIT_COMMIT}
               """
          }
        }
      }
    }


  }
}
