pipeline {
    agent any

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
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
                
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
        }        
    }

    post {
        always {
            junit 'jest-results/junit.xml'
        }
    }
}
