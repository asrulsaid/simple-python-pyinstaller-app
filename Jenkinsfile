node {
    checkout scm
    stage('Build') {
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash name: 'compiled-results', includes: 'sources/*.py*'
        }
    }    
    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            junit 'test-reports/results.xml'
        }
    }
    stage('Deploy') {
        withEnv(["VOLUME=${pwd})/sources:/src", "IMAGE=cdrx/pyinstaller-linux:python2"]) {
            echo "VOLUME is ${VOLUME}"
            echo "IMAGE is ${IMAGE}"
        }
    }
}