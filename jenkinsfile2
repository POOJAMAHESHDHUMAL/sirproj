pipeline {
    agent {
        node {
            label 'built-in'
            customWorkspace '/mnt/war'
        }
    }

    stages {

        stage('SCM') {
            steps {
                sh '''
                rm -rf project
                git clone https://github.com/Hrishikeshkul/project.git
                '''
            }
        }

        stage('BUILD') {
            steps {
                sh '''
                cd /mnt/war/project
                mvn clean package
                '''
            }
        }

        stage('UPLOAD-WAR-TO-S3') {
            steps {
                sh '''
                aws s3 cp \
                /mnt/war/project/target/LoginWebApp.war \
                s3://war-loginwebap/
                '''
            }
        }

        stage('DEPLOY ON SLAVE') {
            agent {
                node {
                    label 'slave1'
                    customWorkspace '/mnt/jenkins-slave1'
                }
            }

            steps {
                sh '''
                sudo yum install -y docker

                sudo systemctl start docker
                sudo systemctl enable docker

                docker rm -f server1 || true

                docker pull tomcat:9

                docker run -d -p 90:8080 --name server1 tomcat:9

                aws s3 cp s3://war-loginwebap/LoginWebApp.war .

                docker cp LoginWebApp.war server1:/usr/local/tomcat/webapps/

                docker restart server1
                '''
            }
        }
    }
}
