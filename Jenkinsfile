def registry = 'https://fqts01rk.jfrog.io/'
pipeline {
    agent {
        node {
            label 'maven'
        }    
    }
    environment {
        PATH = "/opt/apache-maven-3.9.9/bin:$PATH"
        DOCKER_TAG = '2.1.3'
        DOCKER_IMAGE_NAME = 'ttrend-img'
        JFROG_REGISTRY = 'https://fqts01rk.jfrog.io'
        ARTIFACTORY_REPO = 'fqts01rk.jfrog.io/fqts01-docker-local'
    }
    stages {
        stage("build") {
            steps {
                //git branch: 'main', url: 'https://github.com/pranvi13/tweet-trend-for-Project2.git'
                echo "build started"
                sh 'mvn clean deploy'
                echo "build completed"
            }
        }    
            stage('SonarQube analysis') {
            environment {
            scannerHome = tool 'fqts-sonar-scanner'
                   }
            steps{
                withSonarQubeEnv('fqts-sonar-server') { 
                sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        
        }
        stage("Jar Publish") {
          steps {
            script {
                    echo '<--------------- Jar Publish Started --------------->'
                    def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"jenkins-jfrog-cred"
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                    def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "fqts-01rk-libs-release-local/{1}",
                              "flat": "false",
                              "props" : "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
                            }
                        ]
                    }"""
                    def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)
                    echo '<--------------- Jar Publish Ended --------------->'  
            
                }
            }   
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_TAG} .
                    """
                }
            }
        }
        stage('Publish Docker Image to Artifactory') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'jenkins-jfrog-cred', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh """
                    echo $PASSWORD | docker login $JFROG_REGISTRY --username $USERNAME --password-stdin
                    echo '<--------------- docker login done --------------->' 
                    docker tag ${DOCKER_IMAGE_NAME}:${DOCKER_TAG} ${ARTIFACTORY_REPO}/${DOCKER_IMAGE_NAME}:${DOCKER_TAG}
                    docker push ${ARTIFACTORY_REPO}/${DOCKER_IMAGE_NAME}:${DOCKER_TAG}
                    """
                    }
                }
            }
        }
    }
}
