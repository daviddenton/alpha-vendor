#!/usr/local/bin/groovy

def label = "worker-${UUID.randomUUID().toString()}"
def version = "latest"
def region = "eu-west-2"

def buildAndPush(String imageName, String version, String region) {
    stage("Build/push ${imageName} image") {
        container('docker') {
            withCredentials([
                    string(credentialsId: 'aws_account_number', variable: 'awsAccountNumber')
            ]) {
                def imageTag = "${awsAccountNumber}.dkr.ecr.${region}.amazonaws.com/${imageName}:${version}"
                sh "docker build -t ${imageTag} ${imageName}/."

                withAWS(credentials: 'aws_credentials') {
                    sh ecrLogin()
                    sh "docker push ${imageTag}"
                }
            }
        }
    }
}

podTemplate(
        label: label,
        containers: [containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true)],
        volumes: [hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')]) {
    node(label) {
        checkout([$class    : 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false,
                  extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/daviddenton/alpha-vendor']]])
        buildAndPush("vendor-app", version, region)
        buildAndPush("vendor-db", version, region)
        buildAndPush("vendor-configurer", version, region)
        buildAndPush("vendor-blank-config", version, region)
    }
}
