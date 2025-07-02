pipeline {
    agent any

    environment {
        IQ_APPLICATION_ID = 'publicIQTest3__hardeepsonatype'
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
                    def iqStageName
                    if (env.BRANCH_NAME == 'main') {
                        iqStageName = 'build'
                    } else if (env.BRANCH_NAME == 'develop') {
                        iqStageName = 'develop'
                    } else {
                        // Default stage for other branches
                        iqStageName = 'develop'
                    }

                    echo "Using IQ Stage: ${iqStageName} for branch: ${env.BRANCH_NAME}"

                    def jarFile = findFiles(glob: 'target/*.jar')[0].path
                    
                    nexusPolicyEvaluation(
                        iqApplication: env.IQ_APPLICATION_ID,
                        iqStage: iqStageName,
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
