// Define Vault configuration once
        def vaultConfig = [
            vaultUrl: env.VAULT_ADDR,
            vaultCredentialId: 'jenkin'
        ]
        
        // Group related secrets by their purpose
        def testSecrets = [
            [path: 'secret/dev-creds/git-pass', secretValues: [
                [envVar: 'TEST', vaultKey: 'test-git-creds'],
            ]]
        ]

pipeline{
    agent any

    environment {
        VAULT_ADDR = 'http://localhost:8200'
    }

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
                stash includes: 'build/**/*', name: 'build-artifacts' 
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
                script {
                    withVault(configuration: vaultConfig, vaultSecrets: testSecrets) {
                        sh '''
                            echo "$TEST"
                            npm install vercel
                            node_modules/.bin/vercel --version
                        '''
                    }
                }
                unstash 'build-artifacts'
            }
        }
        
    }
    
}