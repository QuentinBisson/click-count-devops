pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        // Build the application using a maven docker containerr
        stage('Build application') {
            agent {
                docker { image 'maven:3.6.1-jdk-8-alpine' }
            }
            steps {
                sh 'mvn clean package'
                stash name: 'clickcount', includes: 'target/*.war'
            }
        }
        // Build the docker image, push it to the registry
        stage('Build and push docker image') {
            steps {
                unstash 'clickcount'
                script {
                    // Get the version from the pom
                    def pom = readMavenPom file: 'pom.xml'
                    imageName = "omegas27/click-count:${pom.version}"

                    // Build the image
                    def image = docker.build('omegas27/click-count')
                
                    // Push it
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        image.push('latest')
                        image.push("${pom.version}")
                    }
                }
            }
        }
        stage('Staging') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        try {
                            // Try to remove the existing clickcount container
                            sh "docker -H ssh://vagrant@staging container inspect -f {{.State.Running}} clickcount"
                            sh "docker -H ssh://vagrant@staging container stop clickcount"
                            sh "docker -H ssh://vagrant@staging container rm clickcount"
                        } catch(e) {
                            // Container does not exist on remote server
                            echo "Container ${imageName} was not found"
                        }
                        // We run the container on the staging server
                        sh "docker -H ssh://vagrant@staging container run -d --name clickcount -p 8080:8080 --network redis-network --restart always registry.hub.docker.com/${imageName}"
                    }
                }
            }
        }
        // Before deploying to production, we await for an ok from the team.
        stage('Production') {
            input {
                message "Do you want to approve the deploy in production?"
                ok "Yes"
            }
            options {
                // Real pipeline would timeout after more time for exploratory tests
                timeout(time: 1, unit: 'HOURS')
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        try {
                            sh "docker -H ssh://vagrant@production container inspect -f {{.State.Running}} clickcount"
                            sh "docker -H ssh://vagrant@production container stop clickcount"
                            sh "docker -H ssh://vagrant@production container rm clickcount"
                        } catch(e) {
                            echo "Container ${imageName} was not found"
                        }
                        // We run the container on the production server
                        sh "docker -H ssh://vagrant@production container run -d --name clickcount -p 8080:8080 --network redis-network --restart always registry.hub.docker.com/${imageName}"
                    }
                }
            }
        }
    }
}