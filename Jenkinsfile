node{
	stage('SCM Checkout') {
		git 'https://github.com/markocrnic/jenkins-maven-test.git'
	}
	stage('Compile-Package'){
		def mvnHome = tool name: 'MVN3', type: 'maven'
		sh "${mvnHome}/bin/mvn package -Dhttp.proxyHost=proxymk.win.vipnet.hr -Dhttp.proxyPort=8080 -Dhttps.proxyHost=proxymk.win.vipnet.hr -Dhttps.proxyPort=8080 -Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true -Dmaven.wagon.http.ssl.ignore.validity.dates=true"
	}
}
