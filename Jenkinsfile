pipeline {

    agent {
        docker { image 'node:14-alpine' }
    }
    stages {
    def server = Artifactory.server "Prod Artifactory"
    def rtMaven = Artifactory.newMavenBuild()
    def buildInfo    
        stage('Clone sources') {
            git branch: 'maven',
                url: 'ssh://git@bitbucket.corp.chartercom.com:7999/cicd/sample.git'
        }
    
        stage('Artifactory configuration') {
            rtMaven.tool = "Maven 3.3.9"
            rtMaven.deployer releaseRepo:'libs-release-local', snapshotRepo:'libs-snapshot-local', server: server
            rtMaven.resolver releaseRepo:'libs-release', snapshotRepo:'libs-snapshot', server: server
        }
    
        stage('Maven build') {
            withEnv(["JAVA_HOME=${ tool 'JDK 1.8.162' }", "PATH+MAVEN=${tool 'Maven 3.3.9'}/bin:${env.JAVA_HOME}/bin"]) {
                buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install'
            }  
        }
    
        stage('SonarQube analysis') {
            def scannerHome = tool 'Sonar 7.9';
            withSonarQubeEnv('Sonar 7.9') { // If you have configured more than one global server connection, you can specify its name
                sh "${scannerHome}/bin/sonar-scanner \
                    -Dsonar.jacoco.reportPaths='target/jacoco.exec' \
                    -Dsonar.projectKey='com.charter.architecture.sample-pipeline' \
                    -Dsonar.projectName='Sample' \
                    -Dsonar.projectVersion=${BUILD_NUMBER} \
                    -Dsonar.sources='src/main/java' \
                    -Dsonar.java.binaries='target/classes' \
                    -Dsonar.tests='src/test/java' \
                    -Dsonar.sourceEncoding='UTF-8'  \
                    -Dsonar.java.source='1.8'"
            }
        }
    
        stage('Deploy to Artifactory') {
            rtMaven.deployer.deployArtifacts buildInfo
        }
    
        stage('Publish build info') {
            server.publishBuildInfo buildInfo
        }
    }
}
