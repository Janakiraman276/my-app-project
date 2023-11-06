node{
   stage('SCM Checkout'){
     git 'https://github.com/manojsb24/my-app.git'
   }
   stage('Compile-Package'){

      def mvnHome =  tool name: 'maven5', type: 'maven'   
      sh "${mvnHome}/bin/mvn clean package"
	  sh 'mv target/myweb*.war target/newapp.war'
   }
   stage('SonarQube Analysis') {
	        def mvnHome =  tool name: 'maven5', type: 'maven'
	        withSonarQubeEnv('sonar') { 
	          sh "${mvnHome}/bin/mvn sonar:sonar"
	        }
	    }
   stage('Build Docker Image'){
   sh 'docker build -t manojsb2409/myweb:0.0.2 .'
   }
   stage('Docker Image Push'){
   withCredentials([string(credentialsId: 'dockerPass', variable: 'dockerPassword')]) {
   sh "docker login -u manojsb2409 -p ${dockerPassword}"
    }
   sh 'docker push manojsb2409/myweb:0.0.2'
   }
   stage('Nexus Image Push'){
   sh "docker login -u admin -p admin123 3.108.254.67:8083"
   sh "docker tag manojsb2409/myweb:0.0.2 3.108.254.67:8083/mano:1.0.0"
   sh 'docker push 3.108.254.67:8083/mano:1.0.0'
   }
   stage('Remove Previous Container'){
	try{
		sh 'docker rm -f tomcattest'
	}catch(error){
		//  do nothing if there is an exception
	}
   stage('Docker deployment'){
   sh 'docker run -d -p 8090:8080 --name tomcattest manojsb2409/myweb:0.0.2' 
   }
}
}



