pipeline{
    agent none
    stages{
        stage("Build"){
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install
                    npm run build
                '''
            }
        }
    }
}