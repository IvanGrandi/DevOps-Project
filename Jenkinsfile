pipeline {
    agent any
    
    environment {
        DOTNET_VERSION = '9.0'
        DOTNET_ROOT = "${WORKSPACE}/.dotnet"
        PATH = "${WORKSPACE}/.dotnet:${env.PATH}"
        REPO_URL = 'https://github.com/jellyfin/jellyfin.git'
        REPO_BRANCH = 'release-10.11.z'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: "${env.REPO_URL}", branch: "${env.REPO_BRANCH}"
            }
        }

        stage('Install .NET') {
            steps {
                echo 'Downloading and installing .NET locally...'
                sh '''
                    mkdir -p $DOTNET_ROOT
                    curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --channel $DOTNET_VERSION --install-dir $DOTNET_ROOT
                '''
            }
        }

        stage('Build') {
            steps {
                sh 'dotnet build --configuration Release'
            }
        }


        stage('Stop Old Process') {
            steps {
                sh 'pkill -f "dotnet run --project Jellyfin.Server" || true'
            }
        }

				// Stage Run WIP
    }


    // Clean up workspace after build
    post {
        always {
            cleanWs()
        }
    }
}
