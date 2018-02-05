

node {
  wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'XTerm', 'defaultFg': 2, 'defaultBg':1]) {
    checkout scm

    stage('Configure and Build') {
      sh "./scripts/rke/configure.sh"
      sh "mkdir -p .ssh && echo \"${AWS_SSH_PEM_KEY}\" > .ssh/${AWS_SSH_KEY_NAME} && chmod 400 .ssh/*"
      sh "./scripts/rke/build.sh"
    }

    stage('Run Validation Tests') {
      try {
        sh "docker run --name ${JOB_NAME}${env.BUILD_NUMBER}  --env-file .env " +
           "rancher-validation-tests /bin/bash -c \'pytest -v -s --junit-xml=results.xml ${PYTEST_OPTIONS} rke_tests/\'"
      } catch(err) {
        echo 'Test run had failures. Collecting results...'
      }

    }

    stage('Test Report') {
      sh "docker cp ${JOB_NAME}${env.BUILD_NUMBER}:/src/rancher-validation/results.xml ."
      step([$class: 'JUnitResultArchiver', testResults: '**/results.xml'])
      sh "docker rm -v ${JOB_NAME}${env.BUILD_NUMBER}"
    }

  }
}