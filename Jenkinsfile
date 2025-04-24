pipeline{
    agent any

    stages{
        stage("Cloning from Github..."){
            steps{
                script{
                    echo "Cloning from Github..."
                    checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-token', url: 'https://github.com/anvesh4161/anime_recommender.git']])
                }
            }
        }
    }
}