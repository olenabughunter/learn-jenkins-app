pipeline {
    agent any

    environment{
        NETLIFY_PROJECT_ID = 'de52fdea-1984-488b-8e78-7743e7b343e6'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Build') {
            agent{
                docker{
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

        stage('Test Running'){
            parallel{

                    stage('Unit Test'){
                        agent{
                            docker{
                                image 'node:18-alpine'
                                reuseNode true
                            }
                        }
                        steps{
                            sh '''
                                echo "Small change"
                                echo 'Test stage'
                                test -f build/index.html
                                npm test
                            '''
                        }
                            post {
                                    always{
                                        junit 'jest-results/junit.xml'
                                    }
                                }
                    }

                    stage('E2E'){
                        agent{
                            docker{
                                image 'mcr.microsoft.com/playwright:v1.57.0-noble'
                                reuseNode true
                            }
                        }
                        steps{
                            sh '''
                                npm install serve
                                node_modules/.bin/serve -s build &
                                sleep 10
                                npx playwright test --reporter=html
                            '''
                        }

                            post {
                                    always{
                                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                                    }

                             }
                    }   
            }

        }

       stage('Deploy Staging') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    node_modules/.bin/netlify link --auth "$NETLIFY_AUTH_TOKEN" --id "$NETLIFY_PROJECT_ID"
                    echo "Deploying to staging. Project-id $NETLIFY_PROJECT_ID"
                    node_modules/.bin/netlify status 
                    node_modules/.bin/netlify deploy --dir=build --no-build --json > deploy-output.json
                '''
            }
        }

        stage('Approval') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                }
            }
        }

        stage('Deploy Prod') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    node_modules/.bin/netlify link --auth "$NETLIFY_AUTH_TOKEN" --id "$NETLIFY_PROJECT_ID"
                    echo "Deploying to production. Project-id $NETLIFY_PROJECT_ID"
                    node_modules/.bin/netlify status 
                    node_modules/.bin/netlify deploy --dir=build --no-build --prod
                '''
            }
        }

        stage('E2E PROD'){
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.57.0-noble'
                    reuseNode true
                }
            }

            environment{
                CI_ENVIRONMENT_URL = 'https://wondrous-douhua-3d52a5.netlify.app'
            }

            steps{
                sh '''
                    npx playwright test --reporter=html
                '''
            }

                post {
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E PROD Report', reportTitles: '', useWrapperFileDirectly: true])
                        }

                    }
        }   


    }
}   