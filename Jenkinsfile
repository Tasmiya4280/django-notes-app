pipeline {
    agent { label 'local-agent' }

    triggers {
        githubPush()
    }

    stages {

        stage("Code Clone") {
            steps {
                script {
                    // Check if .git folder exists in workspace
                    if (!fileExists('.git')) {
                        echo "Repository not found. Cloning..."
                        git branch: 'main', url: 'https://github.com/Tasmiya4280/django-notes-app.git'
                    } else {
                        echo "Repository already exists. Skipping clone."
                    }
                }
            }
        }

        stage("Build Image") {
            steps {
                script {
                    // Check if the Docker image already exists
                    def imageExists = sh(
                        script: "docker images -q my-notes-app",
                        returnStdout: true
                    ).trim()

                    if (imageExists == "") {
                        echo "Docker image not found. Building..."
                        sh "docker build -t my-notes-app ."
                    } else {
                        echo "Docker image already exists. Skipping build."
                    }
                }
            }
        }

        stage("Push") {
            steps {
                echo "Pushing the image to Docker Hub"
                withCredentials([
                    usernamePassword(
                        credentialsId: "dockerhub",
                        usernameVariable: "dockerhubUser",
                        passwordVariable: "dockerhubPass"
                    )
                ]) {
                    sh "docker tag my-notes-app ${env.dockerhubUser}/my-notes-app:latest"
                    sh "docker login -u ${env.dockerhubUser} -p ${env.dockerhubPass}"
                    sh "docker push ${env.dockerhubUser}/my-notes-app:latest"
                }
            }
        }

        stage("Deploy") {
    steps {
        echo "Deploying the Docker image with MySQL"

        sh """
            # Create network if not exists
            docker network inspect my-net >/dev/null 2>&1 || docker network create my-net

            # Stop & remove old containers
            docker stop my-notes-app || true
            docker rm my-notes-app || true

            docker stop mysql-db || true
            docker rm mysql-db || true

            # Run MySQL container
            docker run -d --name mysql-db --network my-net \\
                -e MYSQL_ROOT_PASSWORD=1234 \\
                -e MYSQL_DATABASE=notes \\
                -p 3306:3306 \\
                mysql:5.7

            # Wait for MySQL to initialize
            sleep 20

            # Run Django app container
            docker run -d --name my-notes-app --network my-net -p 8000:8000 \\
                -e DB_NAME=notes \\
                -e DB_USER=root \\
                -e DB_PASSWORD=1234 \\
                -e DB_HOST=mysql-db \\
                tasmiyairfan289/my-notes-app:latest
        """
    }
}

    }
}
// added a new comment to trigger webhook
// 2nd comment
