// Jenkinsfile - Pipeline DevSecOps de "Sistema Alumnos" (OCY1102)
pipeline {
    agent any

    options {
        timestamps()
        timeout(time: 40, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    triggers {
        pollSCM('H/3 * * * *')
    }

    environment {
        PIP_DISABLE_PIP_VERSION_CHECK = '1'
    }

    stages {
        stage('Build') {
            steps {
                script {
                    def sha = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                    env.APP_VERSION = "1.0.${env.BUILD_NUMBER}+${sha}"
                }
                echo "Versión de este build: ${env.APP_VERSION}"
                sh 'git log -1 --format="Commit %h de %an: %s"'
                sh 'rm -rf .venv reports && mkdir -p reports'
                sh 'python3 -m venv .venv'
                sh '.venv/bin/pip install --quiet -r requirements.txt -r requirements-dev.txt'
                sh '.venv/bin/python -m compileall -q app.py'
            }
        }

        stage('Test') {
            steps {
                sh '.venv/bin/python -m pytest --junitxml=reports/junit.xml'
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'reports/junit.xml'
                }
            }
        }

        // >>> [SEGURIDAD]: SAST (Bandit) + SCA (pip-audit)
        stage('Security Review (SAST & SCA)') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
                    sh '''#!/bin/bash
                    set +e
                    .venv/bin/bandit -f json -o reports/bandit.json app.py
                    .venv/bin/bandit -f txt -o reports/bandit.txt app.py
                    cat reports/bandit.txt
                    .venv/bin/bandit -q -ll app.py
                    EXIT_BANDIT=$?

                    .venv/bin/pip-audit -r requirements.txt -f json -o reports/pip-audit.json
                    .venv/bin/pip-audit -r requirements.txt > reports/pip-audit.txt 2>&1
                    EXIT_AUDIT=$?
                    cat reports/pip-audit.txt

                    if [ $EXIT_BANDIT -ne 0 ] || [ $EXIT_AUDIT -ne 0 ]; then
                        echo "Hallazgos de seguridad detectados en SAST o Dependencias."
                        exit 1
                    fi
                    '''
                }
            }
        }

        // >>> [ZAP]: Despliegue en Staging (5001) + Escaneo dinámico OWASP ZAP
        stage('Deploy Staging & DAST (OWASP ZAP)') {
            steps {
                sh 'bash scripts/deploy.sh staging 5001'
                catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
                    sh '.venv/bin/python scripts/zap_scan.py http://jenkins:5001'
                }
            }
        }

        // >>> [GATE]: Solo despliega a Producción si todas las revisiones y escaneos pasaron
        stage('Deploy Producción') {
            when {
                expression { currentBuild.currentResult == 'SUCCESS' }
            }
            steps {
                sh 'bash scripts/deploy.sh produccion 5000'
                script { env.PRODUCCION = env.APP_VERSION }
            }
        }
    }

    post {
        always {
            withEnv(["RESULTADO=${currentBuild.currentResult}"]) {
                sh 'python3 scripts/trazabilidad.py'
            }
            archiveArtifacts artifacts: 'reports/**', allowEmptyArchive: true
        }
    }
}