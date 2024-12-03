pipeline {
    agent any
	tools{
        jdk 'jdk'
        nodejs 'node'
    }

	triggers {
        githubPush() // Trigger the pipeline when a push event occurs
    }
    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker')
        FRONTEND_IMAGE = 'vaibhavnitor/frontend-image'
        BACKEND_IMAGE = 'vaibhavnitor/backend-image'
        DATABASE_IMAGE = 'vaibhavnitor/database-image'
	//SCANNER_HOME=tool 'sonar-scanner'   
    }

    stages {
        stage('Checkout from Git'){
            steps{
                git branch: 'test_secret_sonar', url: 'https://github.com/C2-84177-Assignments/Angular-dotnet-SQL.git'
        }   
     }
	    //stage("Sonarqube Analysis "){
            //steps{
              //  withSonarQubeEnv('sonar-server') {
                //    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=POC \
                  //  -Dsonar.projectKey=POC '''
                //}
            //}
        //}
	  //  stage("quality gate"){
           //steps {
             //   script {
		//	timeout(time: 2, unit:"MINUTES"){
                  //  waitForQualityGate abortPipeline: false, credentialsId: 'Sonar' 
                //}
            //} 
        //}
	//}			
	  //  stage('Install Dependencies') {
          //  steps {
          //      sh "npm install"
          //  }
      //  }
	    stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                script {
                    //Replace 'frontend' with the path to the frontend Dockerfile
                    sh 'docker build -t ${FRONTEND_IMAGE}:latest ElectronicEquipmentAngular'
                }
            }
        }
        stage('Build Database Docker Image') {
            steps {
                script {
                    // Replace 'database' with the path to the database Dockerfile
                    sh 'docker build -t ${DATABASE_IMAGE}:latest .'
                    sh 'docker run -d -p 1433:1433 ${DATABASE_IMAGE}:latest'
                }
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                script {
                    // Replace 'backend' with the path to the backend Dockerfile
                    sh 'docker build -t ${BACKEND_IMAGE}:latest ElectricEquipmentDotNetCoreAPI'
                }
            }
        }
		stage("TRIVY"){
           steps{
            sh "trivy image ${FRONTEND_IMAGE}:latest "
			sh "trivy image ${BACKEND_IMAGE}:latest "
			sh "trivy image ${DATABASE_IMAGE}:latest "
            }
        }
        stage('deploy to container'){
            steps{
                script{
                        sh'docker run -d -p 80:80 ${FRONTEND_IMAGE}:latest'
                        sh'docker run -d -p 81:80 ${BACKEND_IMAGE}:latest'
                }
            }
        }
        stage('Push Images to Docker Hub') {
    steps {
        script {
            withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {          sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
 
                // Push the Docker images
                sh 'docker push ${FRONTEND_IMAGE}:latest'
                sh 'docker push ${BACKEND_IMAGE}:latest'
                sh 'docker push ${DATABASE_IMAGE}:latest'
            }
        }
    }
        
  }
}
}
