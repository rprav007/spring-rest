#!/usr/bin/groovy


node (''){
    env.DEV_PROJECT = env.OPENSHIFT_BUILD_NAMESPACE
    env.SOURCE_CONTEXT_DIR = ""
    env.UBER_JAR_CONTEXT_DIR = "target/"
    env.MVN_COMMAND = "clean deploy"
    env.APP_NAME = "${env.JOB_NAME}".replaceAll(/-?${env.PROJECT_NAME}-?/, '').replaceAll(/-?pipeline-?/, '').replaceAll('/','')
	echo env.APP_NAME
    env.OCP_API_SERVER = "${env.OPENSHIFT_API_URL}"
    env.OCP_TOKEN = readFile('/var/run/secrets/kubernetes.io/serviceaccount/token').trim()
    env.MVN_RELEASE_DEPLOYMENT_REPOSITORY = "nexus::default::http://nexus:8081/repository/maven-releases"
}



node('jenkins-slave-mvn') {

  stage('SCM Checkout') {
    checkout scm
  }

  stage('Build App') {
	sh "mvn ${env.MVN_COMMAND} -DaltDeploymentRepository=${MVN_RELEASE_DEPLOYMENT_REPOSITORY}"
  }
  stage('Build Image') {
	sh "oc start-build ${env.APP_NAME} --from-dir=${env.UBER_JAR_CONTEXT_DIR} --follow"
  }
//  stage('Build Image') {
//	sh "oc start-build ${env.APP_NAME} --from-dir=${env.UBER_JAR_CONTEXT_DIR} --follow"
  //}

  // no user changes should be needed below this point
  stage ('Deploy to Dev') {

    openshiftDeploy (apiURL: "${env.OCP_API_SERVER}", authToken: "${env.OCP_TOKEN}", deploymentConfig: "${env.APP_NAME}")

    openshiftVerifyDeployment (apiURL: "${env.OCP_API_SERVER}", authToken: "${env.OCP_TOKEN}", depCfg: "${env.APP_NAME}", namespace: "${env.DEV_PROJECT}", verifyReplicaCount: true)
  }
}
