pipeline{
    agent any
    environment{
        NETLIFY_SITE_ID = 'f9543224-4148-4f24-9314-77ac6aff055a'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }
    stages{
        stage('build'){
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh '''
                    echo "running build phase"
                    node --version
                    npm --version
                    npm ci
                    npm run build
                '''
            }
        }
        stage('run tests'){
            parallel{
                stage('test'){
                    agent{
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            echo "starting test phase"
                            test -f build/index.html
                            echo "running the test on npm"
                            npm test
                        '''
                    }
                    post{
                        always{
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
                stage('E2E'){
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.49.1-noble'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            npx playwright test --reporter=html
                        '''
                    }
                    post{
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage('deploy staging'){
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh '''
                    npm --version
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    echo "deploying to production. site id : $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                '''
                script{
                    env.staging_url = sh(script:"node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json",returnStdout:true)
                }
            }
        }
        stage('staging E2E'){
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.49.1-noble'
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL="${env.staging_url}"
            }
            steps{
                sh '''
                    npx playwright test --reporter=html
                '''
            }
            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'staginsg E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Approvel'){
            steps{
                timeout(time: 30, unit: 'MINUTES') {
                    input message: 'Ready to deply', ok: 'Deploy'
                }
            }
        }

        stage('deploy prod'){
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh '''
                    npm --version
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "deploying to production. site id : $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
        stage('deploy E2E'){
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.49.1-noble'
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL='https://celadon-sunburst-4cc00c.netlify.app'
            }
            steps{
                sh '''
                    npx playwright test --reporter=html
                '''
            }
            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'prod E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}