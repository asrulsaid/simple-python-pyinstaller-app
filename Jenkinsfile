node {

    stage('Build') {
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        }
    }

    try {
        stage('Test') {
                docker.image('qnib/pytest').inside {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            }
        }
    } catch (e) {
        echo 'Test failed'
        throw e
    } finally {
        junit 'test-reports/results.xml' 
        echo 'Test Success'
    }

}