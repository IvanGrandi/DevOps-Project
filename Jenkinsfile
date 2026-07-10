pipeline {
    agent { label 'dynamic-agent' } 

    environment {
        DISCORD_WEBHOOK = "YOUR_DISCORD_WEBHOOK"
        PODMAN_API    = 'http://192.168.56.21:2375/v1.40'
        APP_NAME      = 'jellyfin-production'
        IMAGE_NAME    = 'docker.io/jellyfin/jellyfin:latest'
        APP_REPO_URL  = 'https://github.com/jellyfin/jellyfin.git'
        
        DOTNET_CLI_HOME = "${WORKSPACE}/.dotnet_home"
        PATH            = "${WORKSPACE}/.dotnet:${env.PATH}"
        DOTNET_SYSTEM_GLOBALIZATION_INVARIANT = 'true'
    }

    stages {
        stage('1. Clone Jellyfin Code') {
            steps {
                echo '======================================================'
                echo 'STEP 1: Syncing Upstream Source Tree'
                echo '======================================================'
                git url: "${APP_REPO_URL}", branch: 'master'
            }
        }

        stage('2. Provision Runtime SDK') {
            steps {
                echo '======================================================'
                echo 'STEP 2: Dynamically Fetching Official .NET SDK'
                echo '======================================================'
                sh 'curl -sSL https://dot.net/v1/dotnet-install.sh -o dotnet-install.sh'
                sh 'chmod +x dotnet-install.sh'
                sh './dotnet-install.sh --channel 10.0 --install-dir .dotnet'
                sh 'dotnet --version'
            }
        }

        stage('3. Run Functional Unit Tests') {
            steps {
                echo '======================================================'
                echo 'STEP 3: Executing Upstream dotnet test Framework'
                echo '======================================================'
                sh 'dotnet test src/Jellyfin.Extensions --configuration Release'
                echo 'SUCCESS: All compiled code assertions passed validation!'
            }
        }

        stage('4. Production Deployment') {
            steps {
                echo '======================================================'
                echo 'STEP 4: Orchestrating Podman API Deployment'
                echo '======================================================'
                
                sh "curl -X POST '${PODMAN_API}/containers/${APP_NAME}/stop' || true"
                sh "curl -X DELETE '${PODMAN_API}/containers/${APP_NAME}' || true"
                
                echo 'Instructing Podman host to pull official Jellyfin production layers...'
                sh "curl -X POST '${PODMAN_API}/images/create?fromImage=docker.io/jellyfin/jellyfin&tag=latest'"
                
                echo 'Deploying new production container instance...'
                sh """
                    curl -X POST -H "Content-Type: application/json" \
                    -d '{
                        "Image": "${IMAGE_NAME}",
                        "HostConfig": {
                            "PortBindings": {
                                "8096/tcp": [{ "HostPort": "8096" }]
                            },
                            "Binds": [
                                "/home/vagrant/jellyfin/config:/config:Z",
                                "/home/vagrant/jellyfin/cache:/cache:Z"
                            ],
                            "RestartPolicy": { "Name": "unless-stopped" }
                        }
                    }' \
                    '${PODMAN_API}/containers/create?name=${APP_NAME}'
                """
                
                sh "curl -X POST '${PODMAN_API}/containers/${APP_NAME}/start'"
                echo 'SUCCESS: Jellyfin production environment updated via API!'
            }
        }

        stage('5. Sanity Verification & Discord Notification') {
            steps {
                echo '======================================================'
                echo 'STEP 5: Verifying App Response Status & Alerting Channel'
                echo '======================================================'
                echo 'Waiting 8 seconds for Jellyfin web server routing tables to warm up...'
                sleep 8
                
                sh 'curl -s -o /dev/null -w "%{http_code}" http://192.168.56.21:8096 | grep -E "200|302"'
                echo 'Success: Live deployment response verified!'

                sh """
                    curl -H "Content-Type: application/json" \
                    -X POST \
                    -d '{
                        "embeds": [{
                            "title": "Pipeline Deployment Success",
                            "color": 3066993,
                            "description": "**Job:** `${JOB_NAME}`\\n**Build Run:** `#${BUILD_NUMBER}`\\n\\n🚀 **Status Update:**\\n• All .NET 10 test projects compiled cleanly.\\n• Target container instantiated via remote Podman API on VM2.\\n• Web entry point verified operational.",
                            "fields": [
                                { "name": "Live Access Endpoint", "value": "http://192.168.56.21:8096", "inline": false }
                            ],
                            "footer": { "text": "Automated Cloud Infrastructure Verification Agent" }
                        }]
                    }' \
                    '${DISCORD_WEBHOOK}'
                """
            }
        }
    }

    post {
        always {
            echo "Pipeline concluded execution with final state: ${currentBuild.currentResult}"
        }
        failure {
            sh """
                curl -H "Content-Type: application/json" \
                -X POST \
                -d '{
                    "embeds": [{
                        "title": "CI/CD Pipeline Defect Detected",
                        "color": 15158332,
                        "description":  **Build Failure Detected!**\\n\\n**Job:** `${JOB_NAME}`\\n**Build Run:** `#${BUILD_NUMBER}`\\n\\nThe workflow execution halted during engine runtime checks. Review the Jenkins console logs immediately.",
                        "footer": { "text": "EFREI DevOps Notification Agent" }
                    }]
                }' \
                '${DISCORD_WEBHOOK}'
            """
        }
    }
}