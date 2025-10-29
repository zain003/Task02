pipeline {
    agent any

    tools {
        // Make sure you have a NodeJS tool named "NodeJS" configured in Manage Jenkins -> Global Tool Configuration
        nodejs 'NodeJS'
    }

    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Branch to build from')
        string(name: 'STUDENT_NAME', defaultValue: 'your name', description: 'Provide your name here — no name, no marks')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'qa', 'prod'], description: 'Select environment')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run Jest tests after build')
    }

    environment {
        // BUILD_NUMBER is available at runtime; Jenkins will expand this value when the pipeline runs.
        APP_VERSION = "1.0.${BUILD_NUMBER}"
        MAINTAINER = "Student"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out branch: ${params.BRANCH_NAME}"
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('temperature-converter') {
                    echo "Installing required packages..."
                    powershell '''
                        Write-Host "Installing npm packages in temperature-converter directory..."
                        npm install
                        if ($LASTEXITCODE -ne 0) {
                            Write-Error "npm install failed"
                            exit 1
                        }
                    '''
                }
            }
        }

        stage('Build') {
            steps {
                dir('temperature-converter') {
                    echo "Building version ${env.APP_VERSION} for ${params.ENVIRONMENT} environment"
                    powershell '''
                        # Create build directory if it doesn't exist
                        if (-not (Test-Path "../build")) {
                            New-Item -ItemType Directory -Path "../build" -Force
                        }
                        
                        # Copy JavaScript files
                        Copy-Item -Path "src/*.js" -Destination "../build" -Force
                        if ($LASTEXITCODE -ne 0) {
                            Write-Warning "No .js files copied"
                        }

                        # Run build script
                        npm run build
                        if ($LASTEXITCODE -ne 0) {
                            Write-Error "Build script failed"
                            exit 1
                        }

                        # Create version file
                        $version = ${env:APP_VERSION}
                        Set-Content -Path "../build/version.txt" -Value "App version: $version"
                        
                        Write-Host "Build completed successfully!"
                    '''
                }
            }
        }

        stage('Test') {
            when {
                expression { return params.RUN_TESTS }
            }
            steps {
                dir('temperature-converter') {
                    echo "Running Jest tests..."
                    powershell '''
                        Write-Host "Running tests..."
                        npm test
                        if ($LASTEXITCODE -ne 0) {
                            Write-Error "Tests failed"
                            exit 1
                        }
                    '''
                }
            }
        }

        stage('Package') {
            steps {
                echo "Creating zip archive for version ${env.APP_VERSION}"
                // Use PowerShell Compress-Archive; on older agents you may need to install PowerShell 5+ or use a zip utility
                bat "powershell -Command \"Compress-Archive -Path build\\* -DestinationPath build_${env.APP_VERSION}.zip -Force\""
            }
        }

        stage('Deploy (Simulation)') {
            steps {
                echo "Simulating deployment of version ${env.APP_VERSION} to ${params.ENVIRONMENT}"
            }
        }
    }

    post {
        always {
            echo "Cleaning up workspace..."
            deleteDir()
        }
        success {
            echo "Pipeline succeeded! Version ${env.APP_VERSION} built and tested."
        }
        failure {
            echo "Pipeline failed! Check console output for details."
        }
    }
}