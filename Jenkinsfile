pipeline {
    agent any
    environment {
         DOCKER_VERSION = getVersion()
        }

    stages{
        stage("Git clone"){
             steps {
                 checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/sathishkumarusk/cicdpythonapp-k8s.git']]])
             }
        }
        stage("Build Mvn package"){
            steps{
                sh 'echo building'
                sh 'mvn clean package'
            }
        }
        
        stage("Build Docker image"){
            steps{
                sh 'docker build . -t sathishkumarusk/webapp:${DOCKER_VERSION}'
            }
    
        }
        stage("docker login"){
            steps{
                withCredentials([usernamePassword(credentialsId: 'dockercred', passwordVariable: 'hub_passwd', usernameVariable: 'hub_username')]) {
                    sh 'docker login -u $hub_username -p $hub_passwd'
                }
                 }  
        }
        stage ("docker push"){
            steps{
                sh 'docker push sathishkumarusk/webapp:${DOCKER_VERSION}'
            }
        }
        stage("deploying in to GKE"){
            steps {
                step([$class: 'KubernetesEngineBuilder', 
                        projectId: "swift-hangar-275604",
                        clusterName: "terrafrom-test-1",
                        location: "us-west2",
                        manifestPattern: "k8s/dev/",
                        credentialsId: "GCP Compute 101",
                        verifyDeployments: true])
            }
        }
        
    }
}

def getVersion(){    
        def imgVersion = sh returnStdout: true, script: 'git rev-parse --verify --short HEAD'
        return imgVersion 
}