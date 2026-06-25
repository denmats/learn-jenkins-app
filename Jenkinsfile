pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'ae89b5a1-494d-44f4-ae1b-e8bdf9a79edc'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.${env.BUILD_NUMBER}"
    }
  
    stages {

        stage('OCI') {
            agent {
                docker {
                    image 'ghcr.io/oracle/oci-cli:latest'
                    args "--entrypoint=''"
                }
            }

             // Bind Jenkins Credentials to OCI CLI supported environment variables
            environment {
                OCI_CLI_USER        = credentials('oci-user-ocid')
                OCI_CLI_TENANCY     = credentials('oci-tenancy-ocid')
                OCI_CLI_FINGERPRINT = credentials('oci-fingerprint')
                OCI_CLI_REGION      = credentials('oci-region')
                OCI_CLI_KEY_CONTENT = credentials('oci-private-key')
                BUCKET_NAME = 'learn-jenkins-202606240748'
            }

            steps {
                sh '''
                    # Verify OCI CLI version
                    oci --version

                    # 1. Dynamically retrieve the Object Storage Namespace for your tenancy
                    NAMESPACE=$(oci os ns get --query "data" --raw-output)
                    echo "Using Namespace: $NAMESPACE"

                    # 2. List the objects inside your bucket
                    oci os object list --bucket-name learn-jenkins-202606240748 --namespace $NAMESPACE

                    echo "Hello in the bucket!" > index.html
                    export bucket_name=$BUCKET_NAME
                    oci os object copy --bucket-name $BUCKET_NAME --destination-bucket $BUCKET_NAME --source-object-name index.html --destination-object-name index-${BUILD_NUMBER}.html --namespace $NAMESPACE
                    echo "Object copied to index-${BUILD_NUMBER}.html in bucket $BUCKET_NAME"

                '''
            }
        }

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

        stage('Tests') {
            parallel {
                stage('Unit tests') {
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

                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "STAGING_URL_TO_BE_SET_IN_SCRIPT"
            }

            steps {
                sh '''
                    
                    netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --site=$NETLIFY_SITE_ID --auth=$NETLIFY_AUTH_TOKEN --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)
                    echo "Staging URL: $CI_ENVIRONMENT_URL"
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }  

        stage('Deploy prod') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "https://keen-mousse-695933.netlify.app"
            }

            steps {
                sh '''
                    node --version
                    npm install netlify-cli
                    netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --site=$NETLIFY_SITE_ID --auth=$NETLIFY_AUTH_TOKEN --prod
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    
    }
}
