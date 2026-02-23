pipeline {
    agent any

    triggers {
        // Trigger every 5 minutes, only on Mondays (1 = Monday in Jenkins cron)
        cron('H/5 * * * 1')
    }

    tools {
        maven 'Maven'    // Must match Maven installation name in Jenkins Global Tool Configuration
        jdk 'JDK17'      // Must match JDK installation name in Jenkins Global Tool Configuration
    }

    stages {

        stage('Checkout') {
            steps {
                echo '>>> Checking out source code...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo '>>> Compiling the project...'
                sh 'mvn clean compile -DskipTests'
            }
        }

        stage('Test & JaCoCo Coverage') {
            steps {
                echo '>>> Running tests and generating JaCoCo coverage report...'
                sh 'mvn test jacoco:report'
            }
            post {
                always {
                    jacoco(
                        execPattern: '**/target/jacoco.exec',
                        classPattern: '**/target/classes',
                        sourcePattern: '**/src/main/java',
                        exclusionPattern: '**/src/test/**'
                    )
                }
            }
        }

        stage('Package - Generate Artifact') {
            steps {
                echo '>>> Packaging application into JAR artifact...'
                sh 'mvn package -DskipTests'
                sh 'ls -lh target/*.jar'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }
    }

    post {
        always {
            junit testResults: '**/target/surefire-reports/*.xml',
                  allowEmptyResults: true
        }
        success {
            echo 'BUILD SUCCESS: Artifact is ready!'
        }
        failure {
            echo 'BUILD FAILED: Check the logs above for errors!'
        }
        cleanup {
            cleanWs()
        }
    }
}
