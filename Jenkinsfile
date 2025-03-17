pipeline {
    agent {
        node {
            label 'maven'
        }
    }

    stages {
        stage('clone-code') {
            steps {
                 git branch: 'main', url: 'https://github.com/Rohit-327/tweet-trend-for-Project2.git'
            }
        }
    }
}
