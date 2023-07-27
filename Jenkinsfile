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
    stage('Manual Approval') {
        input message: 'Lanjutkan ke tahap Deploy?' 
    }
    stage('Deploy') {
        withEnv(["VOLUME=${pwd()}/sources:/src", "IMAGE=cdrx/pyinstaller-linux:python2"]) {
            dir(env.BUILD_ID){
                unstash name: 'compiled-results'
                sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'" 
   
            }
            
            archiveArtifacts "sources/dist/add2vals" 

            sshPublisher(
                publishers: [
                    sshPublisherDesc(
                        configName: 'ec2-server',
                        transfers: [
                            sshTransfer(
                                cleanRemote: false,
                                excludes: '',
                                execCommand: 'ls -la',
                                execTimeout: 120000,
                                flatten: false,
                                makeEmptyDirs: false,
                                noDefaultExcludes: false,
                                patternSeparator: '[, ]+',
                                remoteDirectory: '/home/ubuntu',
                                remoteDirectorySDF: false,
                                removePrefix: '',
                                sourceFiles: 'sources/dist/add2vals'
                            )
                        ], 
                        usePromotionTimestamp: false, 
                        useWorkspaceInPromotion: false, 
                        verbose: false
                    )
                ]
            )

            sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
            sleep 60
        }
    }
}