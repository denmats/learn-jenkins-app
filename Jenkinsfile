pipeline {
    agent any

    stages {
        /*
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        */

        stage('Test stage') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    #test -f build/index.html
                    npm test
                '''
            }
        }

        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install serve
                    node_modules/.bin/serve -s build &
                    sleep 10
                    npx playwright test --reporter=html
                '''
            }
        }
    }

    post {
        always {
            junit 'jest-results/junit.xml'
            junit 'test-results-e2e/junit.xml'

            writeFile file: 'playwright-report/playwright-report-wrapper.html',
            text: '''<!DOCTYPE html>
            <html>
            <head>
            <meta charset="utf-8">
            <title>Playwright Report</title>
            </head>
            <body>
            <iframe src="index.html" sandbox="allow-scripts allow-same-origin allow-forms allow-modals allow-popups" srcdoc="
                <!DOCTYPE html>
                <html>
                <head>
                <meta charset='utf-8'>
                <style>
                    html,body{margin:0;padding:0;height:100%;overflow:hidden}
                    iframe{border:none;width:100%;height:100vh}
                </style>
                </head>
                <body>
                <iframe src='index.html' sandbox='allow-scripts allow-same-origin allow-forms allow-modals allow-popups'></iframe>
                </body>
                </html>
            "></iframe>
            </body>
            </html>'''

            publishHTML([allowMissing: false, 
            alwaysLinkToLastBuild: false, 
            icon: '', 
            keepAll: false, 
            reportDir: 'playwright-report', 
            reportFiles: 'playwright-report-wrapper.html', 
            reportName: 'Playwright HTML Report', 
            reportTitles: '', 
            useWrapperFileDirectly: true])
        }
    }
}
