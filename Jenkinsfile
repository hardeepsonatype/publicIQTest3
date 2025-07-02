pipeline {
    agent any

    environment {
        IQ_SERVER_ID = 'your-iq-server-id'
        IQ_APPLICATION_ID = 'your-app-id'
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
            // After the build is successful, stash the results.
            post {
                success {
                    echo "Stashing the build artifacts..."
                    // Stash any JAR files found in the target directory.
                    // 'build-artifacts' is a custom name for this stash.
                    stash name: 'build-artifacts', includes: 'target/*.jar'
                }
            }
        }

        stage('Sonatype IQ Scan') {
            steps {
                script {
                    // Create a directory to unstash the artifacts into.
                    dir('un-stashed-artifacts') {
                        echo "Unstashing build artifacts..."
                        // Unstash the files saved from the build stage.
                        unstash 'build-artifacts'

                        def iqStageName
                        // This logic works best in a Multibranch Pipeline job.
                        // In a standard Pipeline job, env.BRANCH_NAME will be null.
                        if (env.BRANCH_NAME == 'main') {
                            iqStageName = 'build'
                        } else if (env.BRANCH_NAME == 'develop') {
                            iqStageName = 'develop'
                        } else {
                            // Default stage for other branches or if branch name is not available.
                            iqStageName = 'develop'
                        }
    
                        echo "Using IQ Stage: ${iqStageName} for branch: ${env.BRANCH_NAME}"
    
                        // Now find the JAR file in the current directory.
                        def jarFile = findFiles(glob: '*.jar')[0].path
                        
                        nexusPolicyEvaluation(
                            iqApplication: env.IQ_APPLICATION_ID,
                            iqStage: iqStageName,
                            scanTargets: jarFile
                        )
                    }
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
