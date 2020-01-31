library 'kypseli@master'
def podYaml = libraryResource 'podtemplates/kubectl.yml'
pipeline {
  agent {
    kubernetes {
      label 'workshop-oc-jcasc-update'
      defaultContainer 'jnlp'
      yaml podYaml
    }
  }
  options { 
    buildDiscarder(logRotator(numToKeepStr: '5'))
    preserveStashes(buildCount: 5)
  }
  stages {
    stage('Run Groovy Scripts') {
      agent { label 'master' }
      when { 
        branch 'master'
      }
      steps {
        echo "preparing Jenkins CLI"
        sh 'curl -O http://managed-masters-ops.cje.svc.cluster.local/managed-masters-ops/jnlpJars/jenkins-cli.jar'
        withCredentials([usernamePassword(credentialsId: 'cli-username-token', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          sh """
            alias cli='java -jar jenkins-cli.jar -s \'http://cjoc/cjoc/\' -auth $USERNAME:$PASSWORD'
            cli groovy = < groovy-scripts/k8s-shared-cloud.groovy
            sleep 5
            cli reload-jcasc-configuration
            sleep 5
            cli reload-configuration
            sleep 5
            cli build config-jobs/reprovision-masters/ -f -v
          """
        }
      }
    }
  }
}
