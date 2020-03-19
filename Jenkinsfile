node{
	stage('SCM Checkout') {
		git 'https://github.com/markocrnic/jenkins-maven-test.git'
	}
	stage('Compile-Package'){
		def mvnHome = tool name: 'MVN3', type: 'maven'
		def mvnCmd = "${mvnHome}/bin/mvn -s maven-settings.xml -Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true -Dmaven.wagon.http.ssl.ignore.validity.dates=true"
		sh "${mvnCmd} clean package "
	}
}
