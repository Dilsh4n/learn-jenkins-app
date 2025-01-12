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
                    echo "running inside docker contaienr"
                    ls -la 
                    node --version
                    npm --version
                    npm ci
                    npm run build
                '''
            }
        }
    }
}