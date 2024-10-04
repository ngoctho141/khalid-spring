pipeline {

    agent any

    tools { 
        maven 'demo-maven'
    }
    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql-root-login')
    }
    stages {

        stage('Build with Maven') {
            steps {
                sh 'mvn --version'
                sh 'java -version'
                sh 'mvn clean package -Dmaven.test.failure.ignore=true'
            }
        }

        stage('Packaging/Pushing imagae') {

            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    sh label: '', script: 'docker build -t ngoctho94/springboot .'
                    sh label: '', script: 'docker push ngoctho94/springboot'
                }
            }
        }

        stage('Deploy MySQL to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh label: '', script: 'docker image pull mysql:8.0'
                sh label: '', script: 'docker network create dev || echo "this network exists"'
                sh label: '', script: 'docker container stop khalid-mysql || echo "this container does not exist" '
                sh label: '', script: 'echo y | docker container prune '
                sh label: '', script: 'docker volume rm khalid-mysql-data || echo "no volume"'

                sh label: '', script: 'docker run --name khalid-mysql --rm --network dev -v khalid-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_LOGIN_PSW} -e MYSQL_DATABASE=db_example  -d mysql:8.0 '
                sh label: '', script: 'sleep 20'
                sh label: '', script: 'docker exec -i khalid-mysql mysql --user=root --password=${MYSQL_ROOT_LOGIN_PSW} < script'
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull ngoctho94/springboot'
                sh 'docker container stop khalid-springboot || echo "this container does not exist" '
                sh 'docker network create dev || echo "this network exists"'
                sh 'echo y | docker container prune '

                sh 'docker container run -d --rm --name khalid-springboot -p 8081:8080 --network dev ngoctho94/springboot'
            }
        }
 
    }
    post {
        // Clean after build
        always {
            cleanWs()
        }
    }
}
