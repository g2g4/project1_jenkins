pipeline {
    agent any
    stages {
        stage('delete previous WP on target VM') {
            steps {
                sh '''
                ansible all -i root@$HOST_IP, -m shell -a "docker-compose -f /tmp/docker-compose.yml stop"
		ansible all -i root@$HOST_IP, -m shell -a "docker-compose -f /tmp/docker-compose.yml rm -f"
                '''
            }
        }
	    stage('Clone  repository') { 
            steps { 
                    deleteDir()
                    git url: 'git@github.com:g2g4/project1_docker-compose.git'
            }
        }
	    stage('switch to a predefined  branch') { 
            steps { 
                    sh """
                        git checkout $BRANCH
                    """
            }
        }
	    stage('copy docker-compose.yml to remote VM') {
            steps {
                sh '''
		scp docker-compose.yml root@$HOST_IP:/tmp
                '''
            }
        }
        stage('install WP on target VM') {
            steps {
                sh '''
                ansible all -i root@$HOST_IP, -m shell -a "docker-compose -f /tmp/docker-compose.yml up -d"
                '''
            }
        }
		stage('testing') { 
            options {
                timeout(time: 20, unit: 'SECONDS') 
				retry(3)
            }
			steps { 
                     sh """
			ansible all -i root@$HOST_IP, -m shell -a "docker ps"
			curl http://$HOST_IP:$TEST_PORT/wp-admin/install.php
                    """
            }
        }
    }
    post {
            success {
                slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
            }
            failure {
                slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
            }
        }
}
