pipeline{
    agent any
    environment {
        IMAGE_REPO = "asia.gcr.io/abstract-stream-316904"
        IMAGE_NAME = 'java-cobrand-demo'
        IMAGE_TAG = 'jenkins'
    }

    tools {
        maven 'maven'
        jdk 'JDK8'
        dockerTool 'docker'
    }

    stages {
        stage ('Artifactory Configuration') {
            steps {
                rtServer (
                    id: "https://pawankumar34424.jfrog.io/",
                    url: "https://pawankumar34424.jfrog.io/",
                    credentialsId: "a69e5c17-641b-4e43-83dc-eecf9bea8be0"
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
                sh 'docker tag "my-image:${IMAGE_TAG}" pawankumar34424.jfrog.io/default-docker-virtual/"my-image:${IMAGE_TAG}"'

            }
        }

        stage('Push Image') {
            steps {
                sh 'echo not pushing anything'
               // sh 'docker push pawankumar34424.jfrog.io/default-docker-virtual/"my-image:${IMAGE_TAG}"'
            }
        }

        stage('Deploy Dev') {
            // input{
            //         message "Deploy to Dev?"
            //     }
            steps{
                // sh 'curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl'
                // sh 'chmod +x ./kubectl'
                sh '/tools/kubectl apply -f deployment.yaml'
            }
        }
    }
    // post {
    // always {
    //     emailext body: 'A Test EMail', recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']], subject: 'Test'
	// 		}
	// }
} 
