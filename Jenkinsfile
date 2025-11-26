pipeline {
    agent any

    tools {
        maven 'maven'   // Jenkins tool name from Global Tool Config
    }

    environment {
        GIT_URL = "https://github.com/your-private-repo/project.git"
        CREDS = "git-credentials-id"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: "${GIT_URL}",
                    credentialsId: "${CREDS}"
            }
        }

        stage('Maven Build') {
            steps {
                sh "mvn clean package -DskipTests"
            }
        }

        stage('Detect JAR Name') {
            steps {
                script {
                    env.BUILD_JAR = sh(
                        script: "ls target/*.jar | head -n 1",
                        returnStdout: true
                    ).trim()

                    if (!env.BUILD_JAR?.trim()) {
                        error("❌ No JAR found inside target/")
                    }

                    echo "🔍 Detected JAR file: ${env.BUILD_JAR}"
                }
            }
        }

        stage('Run Application') {
            steps {
                script {

                    echo "Checking if running instance exists..."

                    def pid = sh(
                        script: "pgrep -f app.jar || true",
                        returnStdout: true
                    ).trim()

                    if (pid) {
                        echo "⚠️ App already running with PID ${pid} — stopping..."
                        sh "kill -9 ${pid}"
                        sleep 2
                    } else {
                        echo "No running instance found."
                    }

                    echo "Copying JAR..."
                    sh "cp ${env.BUILD_JAR} app.jar"

                    echo "Starting application..."
                    sh "nohup java -jar app.jar > app.log 2>&1 &"

                    echo "Application started successfully."
                }
            }
        }
    }

    post {
        always { echo "Pipeline completed." }
        success { echo "✅ Pipeline success!" }
        failure { echo "❌ Pipeline failed!" }
    }
}
