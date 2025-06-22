pipeline {
  agent any
  tools { 
        maven 'Maven_3_2_5'  
    }
   stages{
    stage('CompileandRunSonarAnalysis') {
            steps {
		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=aceeinbuggywebapp -Dsonar.organization=aceeinbuggywebapp -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=dc701bec824c52f6e598305851e7219877f90079'
	    }
        }
	   
	   stage('RunSCAAnalysisUsingSnyk') {
            steps {		
				withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
					sh 'mvn snyk:test -fn'
				}
			}
    }
	   stage('Build') { 
            steps { 
               withDockerRegistry([credentialsId: "dockerLogin", url: ""]) {
                 script{
                 app =  docker.build("aceein")
                 }
               }
            }
    }

	stage('Push') {
            steps {
                script{
                    docker.withRegistry('https://926352914341.dkr.ecr.us-west-2.amazonaws.com', 'ecr:us-west-2:aws_credentials') {
                    app.push("latest")
                    }
                }
            }
    	}

	stage('Kubernetes Deployment of aceein Buggy Web Application') {
	   steps {
	      withKubeConfig([credentialsId: 'kubelogin']) {
		  sh('kubectl delete all --all -n devsecops')
		  sh ('kubectl apply -f deployment.yaml --namespace=devsecops')
		}
	      }
   	}

	stage ('wait_for_testing'){
	   steps {
		   sh 'pwd; sleep 180; echo "Application Has been deployed on K8S"'
	   	}
	   }
	   
	stage('RunDASTUsingZAP') {
          steps {
		    withKubeConfig([credentialsId: 'kubelogin']) {
				sh('zap.sh -cmd -quickurl http://$(kubectl get services/aceeinbuggy --namespace=devsecops -o json| jq -r ".status.loadBalancer.ingress[] | .hostname") -quickprogress -quickout ${WORKSPACE}/zap_report.html')
				archiveArtifacts artifacts: 'zap_report.html'
		    }
	     }
       } 
  }
}
