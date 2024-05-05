pipeline {
    agent {
        node {
            label 'python'
        }
    }
    stages {
        stage('Testing') {
            parallel {
                stage('Unit Test') {
                    steps {
                        sh '''
                            export PYTHONPATH=.
                            pytest --junitxml=result-unit.xml test/unit
                        '''
                    }
                }
                stage('Rest Test') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            sh '''
                                java -jar /var/wiremock/wiremock-standalone-3.5.4.jar --port 9090 --root-dir /var/wiremock &
                                export FLASK_APP=app/api.py
                                flask run &
                                sleep 5
                                export PYTHONPATH=.
                                pytest --junitxml=result-rest.xml test/rest
                            '''
                        }
                    }
                }
            }
        }
        stage('Publish Results') {
            steps {
                junit 'result*.xml'
            }
        }
    }
}
