#!/usr/local/bin/groovy

def label = "worker-${UUID.randomUUID().toString()}"

def version = "latest"
def imageName = "dsp"
def region = "eu-west-2"

podTemplate(label: label, containers: [
        containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true)
],
        volumes: [
                hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
        ]) {
    node(label) {

        checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false,
                  extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/daviddenton/alpha-vendor']]])

        withCredentials([
                string(credentialsId: 'aws_ecr_password', variable: 'awsEcrPassword'),
                string(credentialsId: 'aws_account_number', variable: 'awsAccountNumber')
        ]) {
            stage('Build/push vendor images') {
                container('docker') {
                    def imageTag = "${awsAccountNumber}.dkr.ecr.${region}.amazonaws.com/${imageName}:${version}"
                    sh "docker build -t ${imageTag} ."

                    withAWS(credentials:'aws_credentials') {
                        sh ecrLogin()
                        sh "docker push ${imageTag}"
                    }
                }
            }
        }
    }
}
