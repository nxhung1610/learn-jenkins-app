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
        stage {
            // agent {
            //     docker {
            //         image 'node:18-alpine'
            //         reuseNode true
            //     }
            // }
            steps {
                sh '''
                    npm --version
                '''
            }
        }
    }
}