pipeline{
    agent any
        stages{
            
            // Stage 1 is linting
            stage('Lint HTML'){
                steps{
                    sh 'tidy -q -e *.html'
                }
            }

			// Stage 2 is linting
            stage('Lint Dockerfile'){
                steps{
                    sh '''
						docker pull hadolint/hadolint:latest-debian
						pwd
						hadolint --ignore DL3006 Dockerfile
					'''
              }
            }

            // Stage 3 is building a docker image
            stage('Build Docker Image'){
                steps{
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker build --tag=zaincapstone .
						docker image ls
					'''
				    }
                }
            }

            // Stage 4 is pushing the docker image to DockerHub
            stage('Push the Docker Image To Dockerhub'){
                steps{
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
						docker tag zaincapstone syedzaingilani/zaincapstone
						docker push syedzaingilani/zaincapstone
					'''
				    }
                }
            }

            // Stage 5 is setting up Kubernetes Cluster
            stage('Set current kubectl context') {
				steps {
					withAWS(region:'us-east-1', credentials:'aws_credentials') {
						sh '''
							kubectl config use-context arn:aws:eks:us-east-1:110342860649:cluster/microservicesCluster
						'''
					}
				}
			}

			// Stage 6 is deploying the Blue docker container
			stage('Deploy blue container') {
				steps {
					withAWS(region:'us-east-1', credentials:'aws_credentials') {
						sh '''
							kubectl apply -f ./BlueController.json
						'''
					}
				}
			}

			// Stage 7 is deploying the Green docker container
			stage('Deploy green container') {
				steps {
					withAWS(region:'us-east-1', credentials:'aws_credentials') {
						sh '''
							kubectl apply -f ./GreenController.json
						'''
					}
				}
			}

			// Stage 8 is creating the Kubernetes Service and redirect to Blue Service
			stage('Create the service in the cluster, redirect to blue') {
				steps {
					withAWS(region:'us-east-1', credentials:'aws_credentials') {
						sh '''
							kubectl apply -f ./BlueService.json
						'''
					}
				}
			}

			// Stage 9 is wait for the approval from the developer
			stage('Wait user approval'){
				steps{
					input "Ready to redirect the traffic to green?"
				}
			}

			// Stage 10 is create the Kubernetes Service and redirect to Green Service
			stage('Create the service in the cluster, redirect to green') {
				steps {
					withAWS(region:'us-east-1', credentials:'aws_credentials') {
						sh '''
							kubectl apply -f ./GreenService.json
						'''
					}
				}
			}

		}
}
