pipeline{
    agent any
    environment {
        IMAGE_REPO = "asia.gcr.io/abstract-stream-316904"
        IMAGE_NAME = 'java-cobrand-demo'
        IMAGE_TAG = 'jenkins'
    }

    stages {
        
        stage ('Artifactory Configuration') {
            steps {
                rtServer (
                    id: "https://testjfrogd.jfrog.io/",
                    url: "https://testjfrogd.jfrog.io/",
                    credentialsId: "e7c71781-bf24-4c0d-9256-72921cd4d813"
                )
                 rtMavenResolver (
                    id: 'maven-resolver',
                    serverId: 'https://testjfrogd.jfrog.io/',
                    releaseRepo: 'https://testjfrogd.jfrog.io/ui/admin/repositories/local/jenkins-release',
                    snapshotRepo: 'https://testjfrogd.jfrog.io/ui/admin/repositories/local/jenkins-snapshot'
                )  
                 
                rtMavenDeployer (
                    id: 'maven-deployer',
                    serverId: 'https://testjfrogd.jfrog.io/',
                    releaseRepo: 'https://testjfrogd.jfrog.io/ui/admin/repositories/local/jenkins-release',
                    snapshotRepo: 'https://testjfrogd.jfrog.io/ui/admin/repositories/local/jenkins-snapshot',
                    threads: 6,
                    properties: ['BinaryPurpose=Test', 'Team=DevOps-Test']
                )
            }
        }

        stage('Maven build') {
            steps {
                sh "mvn -Dmaven.test.skip=true clean install"
            }
        }
             

        stage('Build Image') {
            steps {
              
              sh 'docker build -t "my-image:${IMAGE_TAG}" .'
              sh 'docker tag "my-image:${IMAGE_TAG}" testjfrogd.jfrog.io/default-docker-virtual/"my-image:${IMAGE_TAG}"'
              sh 'docker push testjfrogd.jfrog.io/default-docker-virtual/"my-image:${IMAGE_TAG}"'
            }
        }

        stage('Push Image') {
            steps {
                withDockerRegistry([ credentialsId: "gcr:abstract-stream-316904", url: "https://asia.gcr.io/abstract-stream-316904" ]) {
                sh 'docker tag my-image:${IMAGE_TAG} ${IMAGE_REPO}/${IMAGE_NAME}:${IMAGE_TAG}'
                sh 'docker push ${IMAGE_REPO}/${IMAGE_NAME}:${IMAGE_TAG}'
                }
                }
            }
        
        stage('Deploy Dev') {
            input{
                    message "Deploy to Dev?"
                }
            steps{
                git url: 'https://github.com/satyakumr/java-cobrand-demo/'
                step([$class: 'KubernetesEngineBuilder', 
                        projectId: "abstract-stream-316904",
                        clusterName: "marriot-demo",
                        zone: "asia-south1-a",
                        manifestPattern: 'k8s/',
                        credentialsId: "k8s-jenkins",
                        verifyDeployments: false])
            }
        }
    }
    post {
    always {
        emailext body: 'A Test EMail', recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']], subject: 'Test'
			}
	}
} 
