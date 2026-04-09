pipeline {
    // Step 1: Define agent and environment variables
    agent any

    environment {
        CONTAINER_ID = ''
        SUM_PY_PATH   = 'C:\\devops-project\\sum.py'
        DIR_PATH      = 'C:\\devops-project'
        TEST_FILE_PATH = 'C:\\devops-project\\test_variables.txt'
    }

    stages {

        // Step 2: Build the Docker image
        stage('Build') {
            steps {
                echo 'Building Docker image...'
                bat "docker build -t sum-image ${DIR_PATH}"
            }
        }

        // Step 3: Run the container and store its ID
        stage('Run') {
            steps {
                echo 'Starting Docker container...'
                script {
                    def output = bat(
                        script: 'docker run -d sum-image',
                        returnStdout: true
                    )
                    def lines = output.split('\n')
                    CONTAINER_ID = lines[-1].trim()
                    echo "Container ID: ${CONTAINER_ID}"
                }
            }
        }

        // Step 4: Test sum.py inside the container
        stage('Test') {
            steps {
                echo 'Running tests...'
                script {
                    def testLines = readFile(TEST_FILE_PATH).split('\n')
                    for (line in testLines) {
                        if (line.trim() == '') continue
                        def vars = line.split(' ')
                        def arg1 = vars[0]
                        def arg2 = vars[1]
                        def expectedSum = vars[2].toFloat()

                        def output = bat(
                            script: "docker exec ${CONTAINER_ID} python /app/sum.py ${arg1} ${arg2}",
                            returnStdout: true
                        )
                        def result = output.split('\n')[-1].trim().toFloat()

                        if (result == expectedSum) {
                            echo "✅ Test passed: ${arg1} + ${arg2} = ${result}"
                        } else {
                            error "❌ Test failed: ${arg1} + ${arg2} = ${result}, expected ${expectedSum}"
                        }
                    }
                }
            }
        }

        // Step 6: Deploy image to DockerHub
        stage('Deploy') {
            steps {
                echo 'Deploying to DockerHub...'
                script {
                    bat "docker login -u stephane723 -p Brice2.01"
                    bat "docker tag sum-image stephane723/sum-image:latest"
                    bat "docker push stephane723/sum-image:latest"
                }
            }
        }
    }

    // Step 5: Always stop and remove the container after pipeline
    post {
        always {
            echo 'Stopping and removing container...'
            script {
                if (CONTAINER_ID != '') {
                    bat "docker stop ${CONTAINER_ID}"
                    bat "docker rm ${CONTAINER_ID}"
                }
            }
        }
    }
}