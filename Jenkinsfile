pipeline {
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
                    def vaultConfig = [
                        disableChildPoliciesOverride: false, 
                        skipSslVerification: true, 
                        timeout: 60, 
                        vaultCredentialId: 'test-role-credential', 
                        vaultUrl: 'http://host.docker.internal:8200'
                    ]
                    
                    def secrets = [
                        [
                            path: 'secret/dev-creds/vercel', 
                            secretValues: [
                                [envVar: 'VERCEL_TOKEN', vaultKey: 'vercel_token']
                            ]
                        ]
                    ]

                    withVault(configuration: vaultConfig, vaultSecrets: secrets) {
                        sh '''
                            npm install vercel
                            node_modules/.bin/vercel --version
                            node_modules/.bin/vercel --token $VERCEL_TOKEN
                            node_modules/.bin/vercel deploy --prod --prebuilt --yes
                        '''
                    }
                }
                unstash 'build-artifacts'
            }
        }
        
    }
    
}