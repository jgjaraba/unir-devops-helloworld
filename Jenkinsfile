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
                        sh '''
                        java -jar /var/wiremock/wiremock-standalone-3.5.4.jar --port 9090 --root-dir /var/wiremock &
                        export FLASK_APP=app/api.py
                        flask run &
                        '''

                        script {
                            sh '''
                                #!/bin/bash
                                max_attempts=10
                                attempt_count=0
                                while ! nc -z localhost 9090 || ! nc -z localhost 5000; do
                                    sleep 1
                                    attempt_count=$((attempt_count + 1))
                                    echo "Checking connection to localhost on port 9090 and 5000 (attempt: $attempt_count)"
                                    if [ $attempt_count -ge $max_attempts ]; then
                                        echo "Service not available after $max_attempts attempts."
                                        exit 1
                                    fi
                                done
                                echo "Services are now available."
                            '''
                        }
                        sh '''
                            export PYTHONPATH=.
                            pytest --junitxml=result-rest.xml test/rest
                        '''
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