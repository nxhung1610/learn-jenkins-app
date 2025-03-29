pipeline{
    agent any
    stages{
        // stage("Build"){
        //     agent {
        //         docker {
        //             image 'node:18-alpine'
        //         }
        //     }
        //     steps {
        //         sh '''
        //             npm run build
        //         '''
        //     }
        // }
        stage ("Test") {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm run build
                    test -f build/index.html
                    npm test
                '''
            }
        }
        stage ("E2E") {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-noble'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install serve
                    node_modules/.bin/serve -s build & sleep 10
                    npx playwright test
                '''
            }
        }
    }
    post {
        always {
            junit 'jest-results/junit.xml'
        }
    }
}