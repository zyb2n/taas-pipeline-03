pipeline {
  agent {
    kubernetes {
      label 'taaspod'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: taas-jenkins-slave
spec:
  containers:
  - name: taas
    image: zyb2n/taastest:1.5
    command:
    - cat
    tty: true
"""
    }
  }


  stages {
    stage('build') {
      steps {
        container('taas') {
          sshagent (credentials: ['taas-ssh']) {
            sh 'inspec version'
//	    sh 'git clone https://github.com/zyb2n/taas-pipeline-03.git /tmp/taas-pipeline-03'
            sh 'aws --version'
         }
        }
      }
   post {
  always {
	archiveArtifacts artifacts: '$BUILD_NUMBER/*/*', fingerprint: true
        // publish html
        publishHTML target: [
            allowMissing: false,
            alwaysLinkToLastBuild: false,
            keepAll: true,
            reportDir: '$BUILD_NUMBER/www',
            reportFiles: '*.index.html',
            reportName: 'TaaS HTML Report'
          ]
        junit "$BUILD_NUMBER/junitreport/*.xml"
  }

    }

}
  }

}
