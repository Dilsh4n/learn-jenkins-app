pipeline{
    agent any
    stages{
        /* stage('build'){
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh '''
                    echo "running inside docker contaienr"
                    ls -la 
                    node --version
                    npm --version
                    npm ci
                    npm run build
                '''
            }
        } */
        stage('test'){
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh '''
                    echo "testing the file existence"
                    test -f build/index.html
                    echo 'running the test on npm'
                    npm  test
                '''
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
                    npx playwright test
                '''
            }
        }
    }
    post{
        always{
            junit 'jest-results/junit.xml'
        }
    }
}