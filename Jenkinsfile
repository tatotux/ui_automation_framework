#!/usr/bin/env groovy
slackChannel = "automation"

pipeline {
    agent {
        docker {
            image "cypress/included:8.6.0"
            args "--net=host --entrypoint='' -u 0:0"
        }
    }
    environment {
        CREDENTIALS = credentials('creds.txt')
    }
    stages{
        stage('Set up environment') {
            steps {
                sleep(5)
                sh 'npm i'
            }
        }
        stage('Cypress execution') {
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        credentials = readFile(file: "${CREDENTIALS}")
                        lines = credentials.readLines()
                        def USERNAME = lines[0]
                        def PASSWORD = lines[1]
                        wrap([$class: "MaskPasswordsBuildWrapper",
                            varPasswordPairs: [
                                [password: USERNAME],
                                [password: PASSWORD],
                            ]]) {
                            timeout(time: 100) {
                                sh "npx cypress run -e '${USERNAME},${PASSWORD},allure=true' --browser ${BROWSER} --config video=${VIDEO}"
                            }
                        }
                    }
                }
            }
        }
        stage('Reporting') {
            steps {
                sh "apt install default-jre allure -y"
                sh './node_modules/.bin/allure generate ./allure-results'
                //allure includeProperties: false, jdk: '', results: [[path: 'allure-results']]
            }
        }
    }
    post {
        always {
            script {
                if (env.VIDEO == 'true') {
                    archiveArtifacts artifacts: 'cypress/videos/*.mp4'
                } else {
                    archiveArtifacts artifacts: 'allure-report'
                }
                sh 'chmod -R 777 *'
            }
        }
        failure {
            archiveArtifacts artifacts: 'cypress/screenshots/*/*.png'
        }
        unstable {
            archiveArtifacts artifacts: 'cypress/screenshots/*/*.png'
        }
    }
}
