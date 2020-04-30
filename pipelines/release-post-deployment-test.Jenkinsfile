pipeline {
    agent any

    stages {
        stage('Setup Test Workspace') {
            steps {
                dir ('tests/end_to_end') { sh 'sh test_setup.sh ${PWD} ${PWD}/config_test_env.sh test${BUILD_NUMBER###}' }
            }
        }
    }
    post {
        always {
            dir ('tests/end_to_end') { sh 'sh test_cleanup.sh ${PWD} ${PWD}/config_test_env.sh test${BUILD_NUMBER###}' }
        }
    }
}
