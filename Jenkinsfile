pipeline {

      agent any
      stages {
            stage('Start Pipeline') {
              steps {
                 echo "This is the first step of the build"
                 echo "Start Build"
                }
           }

           stage('Lint HTML') {
               steps {
                   sh 'tidy -q -e *.html'
                   sh '''docker run --rm -i hadolint/hadolint < Dockerfile'''
               }
          }


          stage('Build image') {
            steps {
                script {
                    dockerImage = docker.build('susmithasusmi13/kubernetes-clusters-demo:lastest')
                    docker.withRegistry('', 'dockerhub') {
                        dockerImage.push()
                    }
                }
            }
          }

          stage('Deploy App') {
            steps {
                      withAWS(credentials: 'aws-cred', region:'us-east-1') {
			sh '''
			aws eks --region us-east-1 update-kubeconfig --name KubernetesCluster
			kubectl apply -f ./kubernetes-config.yml
			'''
                      }
                  }
            }
            stage('Proceed Updating') {
            steps {
                input "Shall we proceed rolling the update?"
            }
            }
            stage('Rolling Update') {
            steps {
                withAWS(credentials: 'aws-cred', region:'us-east-1') {
			sh '''
			aws eks --region us-east-1 update-kubeconfig --name KubernetesCluster
            kubectl get deployments
            kubectl get svc
			'''
            sh '''
			aws eks --region us-east-1 update-kubeconfig --name KubernetesCluster
            kubectl set image deployment/projectcapstone-deploy projectcapstone-pod=susmithasusmi13/kubernetes-clusters-demo:rollingupdate
            kubectl rollout status deployment projectcapstone-deploy
			'''
            sh '''
            aws eks --region us-east-1 update-kubeconfig --name KubernetesCluster
            kubectl get deployments
            kubectl get svc
            '''
            }
            }
            }
      }
}
