/* 
    To build and deploy on Azure please ensure the following:
    1) An Ubuntu 14.04 slave with the label "linux". Must have JDK 7 and Git installed
    2) Set up a Maven tool installer called "maven-3.3"
    3) Set up environment variables for:
         azureHost : the host to use for deployment - from the Azure console
         svchost : the public DNS name the app will be visible on
    4) Credentials with the id of "azure-deployment-id" containing the ftps user:password for deployment

*/
#! groovy

//discard old builds
properties([
   [$class: 'BuildDiscarderProperty',
      strategy: [$class: 'LogRotator', numToKeepStr: '10', artifactNumToKeepStr: '10']
   ]
])

node('java-build-tools && docker') {
    echo "PWD: ${pwd()}"

    stage 'Checkout'
        //from Git, BitBucket, SVN, etc.
        git 'https://github.com/kishorebhatia/game-of-life/PipelineBranch'

    stage 'Compile, Unit'
        //execute build command(s)
        sh 'mvn clean javadoc:javadoc package'

    stage 'Archive'
        //archive JUnit test results
        step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
        //archive JavaDocs
        step([$class: 'JavadocArchiver', javadocDir: 'gameoflife-core/target/site/apidocs/'])
        //archive the WAR
        step([$class: 'ArtifactArchiver', artifacts: 'gameoflife-web/target/*.war'])

    stage 'Functional Test'
        //Browser-based testing with Selenium and Firefox
        sh """
            cd gameoflife-acceptance-tests
            mvn -Plocal -Dwebdriver.base.url=http://localhost:9090 verify
        """

    stage 'Docker Build and Publish'
        //build images from Dockerfile(s)
        def seleniumImage = docker.build('gameoflife-selenium', '.')
        def appImage = docker.build('kishorebhatia/game-of-life', 'gameoflife-web')
        //push to DockerHub
        docker.withRegistry('', 'dockerhub-credentials') {
            appImage.push('latest')
        }
}




