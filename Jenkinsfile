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
  }
}
