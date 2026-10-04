pipeline {
    agent any

    stages {

        // stage('Build') {
        //     agent {
        //         docker {
        //             image 'node:18-alpine'
        //             reuseNode true
        //         }
        //     }

        //     steps {
        //         sh '''
        //             ls -la
        //             node --version
        //             npm --version
        //             npm ci
        //             npm run build
        //             ls -la
        //             echo $(pwd)
        //             test -f build/index.html
        //         '''
        //     }
        // }

        stage('Run Tests'){
            parallel {
                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            echo "Running Unit Tests"
                            npm ci
                            CI=true npm test
                        '''
                    }
                }
            stage('Playwright Tests') {
                agent {
                    docker {
                        image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                        reuseNode true
                    }
                }

                steps {
                    sh '''
                        echo "Running Playwright Tests"
                        npm ci
                        npx playwright test
                        npx playwright test --reporter=html
                    '''
                }
            }
        }

        // stage('Test') {
        //     agent {
        //         docker {
        //             image 'node:18-alpine'
        //             reuseNode true
        //         }
        //     }

        //     steps {
        //         sh '''
        //             echo "Test Stage"
        //             npm ci
        //             CI=true npm test
        //         '''
        //     }
        // }

        // stage('E2E') {
        //     agent {
        //         docker {
        //             image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
        //             reuseNode true
        //         }
        //     }

        //     steps {
        //         sh '''
        //             echo "E2E Test Stage"
        //             npm install serve
        //             node_modules/.bin/serve -s build -l 3000 &
        //             sleep 10 
        //             npx playwright test
        //             npx playwright test --reporter=html
        //         '''
        //     }
        // }
    }

    post {
        always {
            junit 'jest-results/junit.xml, playwright-results/junit.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
            
        }
    }
}