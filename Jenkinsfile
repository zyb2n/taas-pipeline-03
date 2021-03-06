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
  annotations:
    iam.amazonaws.com/role: arn:aws:iam::884956725745:role/taas-NodeInstanceRole-1P30OR4Q5QLT3
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
    environment {
        AWS_REGION = 'us-east-1'
    }


  stages {
    stage('build') {
      steps {
        container('taas') {
          sshagent (credentials: ['taas-ssh']) {
            sh 'inspec version'
	    sh 'git clone https://github.com/zyb2n/taas-pipeline-03.git /tmp/taas-pipeline-03'
            sh "inspec exec /tmp/taas-pipeline-03/profile-aws -t aws://  --reporter cli json:$BUILD_NUMBER/json/aws-`aws sts get-caller-identity --output text --query 'Account'`.output.json junit:$BUILD_NUMBER/junitreport/aws-`aws sts get-caller-identity --output text --query 'Account'`.junit.xml html:$BUILD_NUMBER/www/aws-`aws sts get-caller-identity --output text --query 'Account'`.index.html || true"
            sh "/es_loader.sh store-elasticsearch-client $BUILD_NUMBER/json/aws-*.output.json"
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
