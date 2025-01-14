pipeline{
    agent any
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
        }
    }
}