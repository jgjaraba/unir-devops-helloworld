pipeline {
    agent {
        node {
            label 'python'
        }
    }

    environment {
        PATH = "/opt/venv/bin:$PATH"
        PYTHONPATH = "."
        FLASK_APP= "app/api.py"
    }

    stages{
        stage('Unit') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS') {
                    sh '''
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
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    cobertura autoUpdateHealth: false,
                    autoUpdateStability: false,
                    coberturaReportFile: 'coverage.xml',
                    onlyStable: false,
                    failUnstable: true,
                    failUnhealthy: true,
                    conditionalCoverageTargets: '100,80,90',
                    lineCoverageTargets: '100,85,95'
                }
            }
        }

        stage('Rest') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS') {
                    script{
                        startFlaskService()
                        startWireMockService()
                        waitForService(5000)
                        waitForService(9090)
                    }
                    sh '''
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
                    if (!isServiceUp(5000)) {
                        startFlaskService()
                        waitForService(5000)
                    }
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
def startFlaskService() {
    sh "flask run &"
}
def startWireMockService() {
    sh "java -jar /var/bin_home/wiremock-standalone-3.5.4.jar --port 9090 --root-dir test/wiremock &"
}
def isServiceUp(port) {
        response = sh(script: "nc -z localhost ${port}", returnStatus: true)
        return response == 0
}
def waitForService(port) {
    timeout(time: 5, unit: 'SECONDS') {
        waitUntil {
            script {
                return isServiceUp(port)
            }
        }
    }
}