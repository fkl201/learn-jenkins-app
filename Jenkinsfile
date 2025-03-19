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
    }
}
