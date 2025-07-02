// Define a variable to hold the SCM information, accessible across stages.
def scmVars

pipeline {
    agent any

    environment {
        IQ_SERVER_ID = 'your-iq-server-id'
        IQ_APPLICATION_ID = 'publicIQTest3__hardeepsonatype'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                script {
                    // The checkout step returns a map of SCM variables, including the branch.
                    // We store this in the scmVars variable for later use.
                    scmVars = checkout scm
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

                        // Determine the branch name from the checkout information.
                        // The GIT_BRANCH variable might be 'origin/main', so we take the last part.
                        def branchName = scmVars.GIT_BRANCH.split('/').last()
                        
                        def iqStageName
                        if (branchName == 'main') {
                            iqStageName = 'build'
                        } else if (branchName == 'develop') {
                            iqStageName = 'develop'
                        } else {
                            // Default stage for other branches.
                            iqStageName = 'develop'
                        }
    
                        echo "Using IQ Stage: ${iqStageName} for branch: ${branchName}"
    
                        // Now find the JAR file recursively in the current directory.
                        // The '**/' pattern searches in all subdirectories.
                        def jarFile = findFiles(glob: '**/*.jar')[0].path
                        
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



