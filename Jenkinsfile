pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '53261136-50fc-404b-a35f-d878be8a7ccb'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {        
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Build stage"
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage ('Run Tests') {
            parallel {
                stage('Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo "Test Stage"
                            test -f build/index.html
                            test -f src/App.js
                            test -f src/App.test.js
                            if [ -f build/index.html ]
                            then
                                echo "build/index.html exists."
                                npm test
                            else
                                echo "build/index.html does not exist."
                            fi
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }                      
                }
                stage('E2E') {
                    agent {
                        docker {
                            //image 'mcr.microsoft.com/playwright:v1.51.1-noble'
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                            //args '-u root:root' (Don't use root)
                        }
                    }
                    steps {
                        sh '''
                            echo "E2E Stage"
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }                    
                }        
            }
        }

        stage('Deploy-Staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploy to Staging SITE_ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status node-jq
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                    node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json
                '''
                script {
                    env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
                }
            }
        }

        stage('Staging E2E') {
            agent {
                docker {
                    //image 'mcr.microsoft.com/playwright:v1.51.1-noble'
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                    //args '-u root:root' (Don't use root)
                }
            }

            environment {
                CI_ENVIRONMENT_URL = '${env.STAGING_URL}'
            }

            steps {
                sh '''
                    echo "Staging E2E Stage"
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }  
        }

        stage('Approve') {
            steps {
                echo 'Waiting for Approval...'
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                }
            }
        }            

        stage('Deploy-Prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploy to Prodution SITE_ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }

        stage('Prod E2E') {
            agent {
                docker {
                    //image 'mcr.microsoft.com/playwright:v1.51.1-noble'
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                    //args '-u root:root' (Don't use root)
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://symphonious-kashata-26b2d8.netlify.app'
            }

            steps {
                sh '''
                    echo "Prod E2E Stage"
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }                    
        }        
    }          
}
