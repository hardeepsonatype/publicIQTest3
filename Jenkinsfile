pipeline {
    agent any

    environment {
        IQ_APPLICATION_ID = 'Publictestiq2'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                script {
                    checkout scm
                }
            }
        }

        stage('Build with Maven') {
            agent {
                docker {
                    image 'maven:3.8.4-openjdk-11'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                script {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Sonatype IQ Scan') {
            steps {
                script {
                    def jarFile = findFiles(glob: 'target/*.jar')[0].path
                    
                    nexusPolicyEvaluation(
                        iqApplication: env.IQ_APPLICATION_ID,
                        iqStage: 'build',
                        scanTargets: jarFile
                    )
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
