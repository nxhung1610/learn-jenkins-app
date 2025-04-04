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
            }
        }
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
                                [envVar: 'VERCEL_TOKEN', vaultKey: 'vercel_token'],
                                [envVar: 'VERCEL_PROJECT_ID', vaultKey: 'vercel_project_id'],
                                [envVar: 'VERCEL_ORG_ID', vaultKey: 'vercel_org_id']
                            ]
                        ]
                    ]

                    withVault(configuration: vaultConfig, vaultSecrets: secrets) {
                        sh '''
                            echo $VERCEL_PROJECT_ID
                            echo $VERCEL_ORG_ID
                            npm install vercel
                        '''
                        sh '''
                            cat <<EOF >> project.json
                            {"projectId":"$VERCEL_PROJECT_ID","orgId":"$VERCEL_ORG_ID"}
                            EOF
                        '''
                        sh 'node_modules/.bin/vercel deploy --local-config project.json --prod --token $VERCEL_TOKEN --yes'
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
        cleanup{
            deleteDir()
        }
    }
    
}