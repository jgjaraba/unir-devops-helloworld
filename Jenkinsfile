pipeline {
    agent {
        node {
            label 'python'
        }
    }

    environment {
        PATH = "/opt/venv/bin:$PATH"
    }
    stages{
        stage('Unit') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS') {
                    sh '''
                        export PYTHONPATH=.
                        coverage run --branch --source=app --omit=app/__init__.py,app/api.py -m pytest --junitxml=result-unit.xml test/unit
                    '''
                    junit 'result-unit.xml'
                }
            }
        }

        stage("Coverage"){
            steps{
                sh '''
                    coverage xml
                '''
                catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
                    cobertura autoUpdateHealth: false,
                    autoUpdateStability: false,
                    coberturaReportFile: 'coverage.xml',
                    onlyStable: false,
                    failUnstable: true,
                    failUnhealthy: false,
                    conditionalCoverageTargets: '100,80,90',
                    lineCoverageTargets: '100,85,95'
                }
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    cobertura autoUpdateHealth: false,
                    autoUpdateStability: false,
                    coberturaReportFile: 'coverage.xml',
                    onlyStable: false,
                    failUnstable: false,
                    failUnhealthy: true,
                    conditionalCoverageTargets: '100,80,90',
                    lineCoverageTargets: '100,85,95'
                }
            }
        }

        stage('Rest') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS') {
                    sh '''
                        java -jar /var/bin_home/wiremock-standalone-3.5.4.jar --port 9090 --root-dir test/wiremock &
                        export FLASK_APP=app/api.py
                        flask run &
                        sleep 5
                        export PYTHONPATH=.
                        pytest --junitxml=result-rest.xml test/rest
                    '''
                    junit 'result-rest.xml'
                }
            }
        }

        stage("Static"){
            steps{
                sh '''
                    flake8 --exit-zero --format=pylint app >flake8.out
                '''
                recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold: 8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unstable: false]]
            }
        }

        stage("Security"){
            steps{
                sh '''
                    bandit --exit-zero -r  . -f custom -o bandit.out --severity-level medium --msg-template="{relpath}:{line}: [{test_id}] {msg}"
                '''
                recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[threshold: 2, type: 'TOTAL', unstable: true], [threshold: 4, type: 'TOTAL', unstable: false]]
            }
        }

        stage("Performance"){
            steps{
                script {
                        sh '''
                            #!/bin/bash
                            if ! nc -z localhost 5000; then
                                flask run &
                                sleep 5
                            fi
                        '''
                        }
                sh "/var/bin_home/apache-jmeter-5.6.3/bin/jmeter -n -t test/jmeter/flask.jmx -f -l flask.jtl"
                perfReport sourceDataFiles: "flask.jtl"
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
