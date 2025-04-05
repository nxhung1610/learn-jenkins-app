pipeline {
    agent any

    stages{
        stage("Build"){
            steps {
                sh '''
                apt update
                apt install npm
                '''
            }
        }
        stage ("Unit Tests") {
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
                            export VERCEL_PROJECT_ID=$VERCEL_PROJECT_ID
                            export VERCEL_ORG_ID=$VERCEL_ORG_ID
                            node_modules/.bin/vercel deploy --prod --token $VERCEL_TOKEN --yes
                        '''
                    }
                }
            }
        }
    }
    
}