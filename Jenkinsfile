
pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '27cb1f9a-55c4-4470-86b3-534ea80d5260'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = '1.2.3'
    }

    stages {

        stage('Docker') {
            steps {
                sh 'docker build -t my-playwright .'
            }
        }

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

        stage('Tests') {
            parallel {

                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            npm test
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
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            npm install --no-save serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: false,
                                keepAll: false,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Local E2E',
                                reportTitles: '',
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
                }
            }
        }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'node:18'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    npm install --no-save netlify-cli
                    node_modules/.bin/netlify --version

                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"

                    node_modules/.bin/netlify deploy \
                        --site "$NETLIFY_SITE_ID" \
                        --dir=build \
                        --json > deploy-output.json
                '''

                script {
                    env.STAGING_URL = sh(
                        script: "node -p \"require('./deploy-output.json').deploy_url\"",
                        returnStdout: true
                    ).trim()

                    echo "Staging URL: ${env.STAGING_URL}"
                }
            }
        }

        stage('Staging E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            steps {
                withEnv(["CI_ENVIRONMENT_URL=${env.STAGING_URL}"]) {
                    sh '''
                        echo "Testing staging: $CI_ENVIRONMENT_URL"
                        npx playwright test --reporter=html
                    '''
                }
            }

            post {
                always {
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: false,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Staging E2E',
                        reportTitles: '',
                        useWrapperFileDirectly: true
                    ])
                }
            }
        }

        stage('Deploy prod') {
            agent {
                docker {
                    image 'node:18'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    npm install --no-save netlify-cli
                    node_modules/.bin/netlify --version

                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"

                    node_modules/.bin/netlify deploy \
                        --site "$NETLIFY_SITE_ID" \
                        --dir=build \
                        --prod \
                        --json > deploy-prod-output.json
                '''

                script {
                    env.PROD_URL = sh(
                        script: "node -p \"require('./deploy-prod-output.json').url\"",
                        returnStdout: true
                    ).trim()

                    echo "Production URL: ${env.PROD_URL}"
                }
            }
        }

        stage('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            steps {
                withEnv(["CI_ENVIRONMENT_URL=${env.PROD_URL}"]) {
                    sh '''
                        echo "Testing production: $CI_ENVIRONMENT_URL"
                        npx playwright test --reporter=html
                    '''
                }
            }

            post {
                always {
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: false,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Prod E2E',
                        reportTitles: '',
                        useWrapperFileDirectly: true
                    ])
                }
            }
        }
    }
}
