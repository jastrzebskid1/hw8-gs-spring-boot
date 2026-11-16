pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }
    tools {
        maven '3.9.11'
    }
    environment {
        // Nexus runs as the "nexus" container on the same Docker network as Jenkins
        NEXUS_URL  = 'http://nexus:8081'
        NEXUS_REPO = 'maven-snapshots'
    }
    stages {
        stage('Checkout Source Code') {
            steps {
                git branch: 'main', url: 'https://github.com/jastrzebskid1/hw8-gs-spring-boot.git'
            }
        }
        stage('Test') {
            steps {
                sh 'git --version'
                sh 'mvn --version'
                sh 'mvn clean test'
            }
        }
        stage('Build and Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
        stage('Push Jar to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-creds',
                                                  usernameVariable: 'NEXUS_USER',
                                                  passwordVariable: 'NEXUS_PASS')]) {
                    sh '''
cat > nexus-settings.xml <<'EOF'
<settings>
  <servers>
    <server>
      <id>nexus</id>
      <username>${env.NEXUS_USER}</username>
      <password>${env.NEXUS_PASS}</password>
    </server>
  </servers>
</settings>
EOF
mvn -s nexus-settings.xml deploy -DskipTests -DaltDeploymentRepository=nexus::${NEXUS_URL}/repository/${NEXUS_REPO}/
'''
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Build successful! Jar pushed to Nexus.'
        }
        failure {
            echo 'Build failed!'
        }
    }
}
