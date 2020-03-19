
	node {

		def APP_NAME = 'jenkins-maven-test'
		def PROJECT_NAME = 'open-api'
		def GIT_REPO = 'https://github.com/markocrnic/jenkins-maven-test.git'
		def IMAGE = 'lvpmkrhsat.onevip.mk:5000/one_vip-ocp311-redhat-openjdk-18_openjdk18-openshift'
		def GIT_SECRET = 'git-mk'
		def ENV_VARIABLES = ' ESB_BRIAN_HEALTHCHECK=http://open-api-brian-esb-consumer.open-api.svc.cluster.local:8080 ESB_BRIAN_CUSTOMERBILL=http://open-api-brian-esb-consumer.open-api.svc.cluster.local:8080/customerbill ESB_BRIAN_QUERYACCOUNT=http://open-api-brian-esb-consumer.open-api.svc.cluster.local:8080/account DB_OPENAPI_HEALTHCHECK=http://open-api-db-log.open-api.svc.cluster.local:8080 DB_OPENAPI_ACCOUNTTYPE=http://open-api-db-log.open-api.svc.cluster.local:8080/getCustomerBill'
		def BUILD_ENV_VARIABLES = ' --build-env ESB_BRIAN_HEALTHCHECK=http://open-api-brian-esb-consumer.open-api.svc.cluster.local:8080 --build-env ESB_BRIAN_CUSTOMERBILL=http://open-api-brian-esb-consumer.open-api.svc.cluster.local:8080/customerbill --build-env ESB_BRIAN_QUERYACCOUNT=http://open-api-brian-esb-consumer.open-api.svc.cluster.local:8080/account --build-env DB_OPENAPI_HEALTHCHECK=http://open-api-db-log.open-api.svc.cluster.local:8080 --build-env DB_OPENAPI_ACCOUNTTYPE=http://open-api-db-log.open-api.svc.cluster.local:8080/getCustomerBill'
		def CREATE_ROUTE = 'true'
		
		def mvnHome = tool name: 'MVN3', type: 'maven'
		def mvnCmd = "${mvnHome}/bin/mvn -s maven-settings.xml -Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true -Dmaven.wagon.http.ssl.ignore.validity.dates=true"

		
		// Checkout Source Code
		stage('Checkout Source') {
		
		  //source is in same repo as Jenkinsfile
		  git "${GIT_REPO}"
		  
		}
		
		// Extract version and other properties from the pom.xml
		  def groupId    = getGroupIdFromPom("pom.xml")
		  echo "groupId: ${groupId}"
		  def artifactId = getArtifactIdFromPom("pom.xml")
		  echo "artifactId: ${artifactId}"
		  def version    = getVersionFromPom("pom.xml")
		  def pomVersion    = "${version}"
		  echo "pomVersion: ${pomVersion}"
		  
		// Get version
		  if (getVersionFromPom("pom.xml").toString().toUpperCase().contains("SNAPSHOT")) {
			version = getVersionFromPom("pom.xml").toString().toUpperCase().replace("-SNAPSHOT", "")
		  } else {
			//If the condition is false print the following statement
			version = getVersionFromPom("pom.xml").toString();
			println("versionInfo without snapshot extension :" +version);
		  }
		  echo "version:  ${version}"
		
		environment {
			
			echo "Setting environment variables."
		
			// If needed for testing, set environment variables
			
			//	## Example of how to set ENV variables ##
			//	DISABLE_AUTH = 'true'
			//	DB_ENGINE    = 'sqlite'
			//	## Example of how to set ENV variables ##
			
		}
		
		// Using Maven build the jar file
		// Do not run tests in this step
		stage('Build Application Binary') {
		
			echo "Building jar file"
			sh "${mvnCmd} clean package -DskipTests=true"
			
		}
		
		// Using Maven run the unit tests
		stage('Unit Tests') {
		
			echo "Running Unit Tests"
			sh "${mvnCmd} test -DskipTests=false"
			
		}
		
		// Publish the built jar file to JFrog
		stage('Publish to JFrog') {
			
			// If defined later, should publis built jar file to JFrog artifactory
			
			echo "Publish to JFrog"
			
			// ## Example of how to publish to artifactory for storing generated jar files
			// sh "${mvnCmd} deploy:deploy-file -DgroupId=${groupId} -DartifactId=${artifactId} -Dversion=${devTag} -Dpackaging=jar -DrepositoryId=nexus -Durl=http://nexus3.cicd.svc.cluster.local:8081/repository/releases -Dfile=target/${artifactId}-${pomVersion}.jar -DskipTests=true -DpomFile=pom.xml"
			// ## Example of how to publish to artifactory for storing generated jar files
		
		}
		
		// Create or replace Image builder artifacts
		stage('Create Image Builder') {
		
			echo "Creating Image Builder"
			sh "if oc get bc ${APP_NAME} --namespace=cicd; \
				then echo \"exist\"; \
				else oc new-build --binary=true --name=${APP_NAME} ${IMAGE} --labels=app=${APP_NAME} -n cicd;fi"
		
		}
		
		// Build the OpenShift Image in OpenShift.
		stage('Build and Tag OpenShift Image') {
		
			echo "Building OpenShift container image ${APP_NAME}"

			// Start Binary Build in OpenShift CICD project using the file we just published
			sh "oc start-build ${APP_NAME} --follow --from-file=target/${APP_NAME}-0.0.1-SNAPSHOT.jar -n cicd"
			echo "oc start build complete."
		
		}
		
		// Copy image to needed project
		stage('Copy Image to Project'){
        openshift.withCluster(){

          // Copy the image to the registry on the non-prod cluster to the target namespace
		  // Maybe should add :latest tag
          // sh "cp docker://docker-registry.default.svc:5000/cicd/${APP_NAME}:latest docker://docker-registry.default.svc:5000/${PROJECT_NAME}/${APP_NAME}:latest"
		  // sh "oc import-image ${APP_NAME} --from=docker-registry.default.svc:5000/cicd/${APP_NAME}:latest"
		  
		  // sh "oc policy add-role-to-user \
		  //		system:image-puller system:serviceaccount:${PROJECT_NAME}:default \
		  //		--namespace=cicd"
		}
		
		// Deploy the built image.
		stage('Configure Deployment') {
			echo "Deploying container image"
			openshift.withCluster(){

			  // Switch to target project on remote cluster
			  openshift.withProject( "${PROJECT_NAME}" ) {
				echo "Using project: ${openshift.project()}"

				//Create application if it does not exist
				def deploymentSelector = openshift.selector( "dc", "${APP_NAME}")
				def deploymentExists = deploymentSelector.exists()
				if (!deploymentExists) {
				  echo "Deployment ${APP_NAME} does not exist"
				  openshift.newApp("${APP_NAME}:latest", "--name=${APP_NAME}", "--allow-missing-imagestream-tags=true","--namespace=${PROJECT_NAME}")
				}
				echo "Deployment ${APP_NAME} exists"

				// Create an external facing route if required
				if ("${CREATE_ROUTE}" == "true") {
					if(openshift.selector("route", APP_NAME).exists()) {
						echo "Route already exposed."
					}else{
						sh 'oc expose service ' + APP_NAME + ' -n ' + PROJECT_NAME
					}
				}

				//Update the Image on the Development Deployment Config
				openshift.raw("set","image dc/${APP_NAME}","${APP_NAME}=${APP_NAME}:latest","--source=imagestreamtag",
						"-n ${PROJECT_NAME}")

			  }
			}
		  }
		}
	}


// Convenience Functions to read variables from the pom.xml
// Do not change anything below this line.
def getVersionFromPom(pom) {
  def matcher = readFile(pom) =~ '<version>(.+)</version>'
  matcher ? matcher[0][1] : null
}
def getGroupIdFromPom(pom) {
  def matcher = readFile(pom) =~ '<groupId>(.+)</groupId>'
  matcher ? matcher[0][1] : null
}
def getArtifactIdFromPom(pom) {
  def matcher = readFile(pom) =~ '<artifactId>(.+)</artifactId>'
  matcher ? matcher[0][1] : null
}
