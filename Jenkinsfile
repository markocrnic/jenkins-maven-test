node{
	stage('SCM Checkout') {
		git 'https://github.com/markocrnic/jenkins-maven-test.git'
	}
	stage('Compile-Package'){
		def mvnHome = tool name: 'MVN3', type: 'maven'
		sh "${mvnHome}/bin/mvn package -DsocksProxyHost=proxymk.win.vipnet.hr -DsocksProxyPort=8080"
	}
}
