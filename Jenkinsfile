pipeline{
    agent any
    stages{
        stage("Build"){
            agent {
                docker {
                    image 'node:18-alpine'
                }
            }
            steps {
                sh '''
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
                stash name: 'build-artifacts', includes: './build'
            }
        }
        stage ("Tests") {
            parallel {
                stage ("Unit Tests") {
                    agent {
                        docker {
                            image 'node:18-alpine'
                        }
                    }
                    steps {
                        sh '''
                            npm run build
                            test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                           junit 'jest-results/junit.xml'
                        }
                    }
                }
                stage ("E2E") {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build & sleep 2
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
                
            }
        }

        stage ("Deploy") {
            agent {
                docker {
                    image 'node:18-alpine'
                }
            }
            steps {
                sh '''
                    npm install vercel -g
                    vercel --version
                '''
                unstash 'build-artifacts'
            }
        }
        
    }
    
}