def appList = [
    "adservice",
    "cartservice",
    "checkoutservice",
    "currencyservice",
    "emailservice",
    "frontend",
    "loadgenerator",
    "paymentservice",
    "productcatalogservice",
    "recommendationservice",
    "shippingservice"
]

pipeline {
    agent any
    stages {
        stage('Prepare') {
            steps {
                script {
                    sh """
                        git config --global user.name 'jenkins-user'
                        git config --global user.email 'eldarsagirovda@github.com'
                    """
                    appList.each   {
                        sh "aws ecr describe-repositories --repository-names esagirov-aws-bb/${it} || aws ecr create-repository --repository-name esagirov-aws-bb/${it}"
                    }
                }
            }
            
        }
        stage('Build') {
            steps { 
                script {
                    appList.each {
                        if (it == "cartservice") {
                            appPath = "src/${it}/src"
                        } else {
                            appPath = "src/${it}"
                        }
                        dir("${appPath}") {
                            sh "docker build . -t 589295909756.dkr.ecr.eu-west-1.amazonaws.com/esagirov-aws-bb/${it}:${BUILD_NUMBER}"
                        }
                    }
                }
            }
        }
        stage('Publish') {
            steps { 
                script {
                    appList.each {
                        dir("src/${it}") {
                            sh "aws ecr get-login-password --region eu-west-1 | docker login --username AWS --password-stdin 589295909756.dkr.ecr.eu-west-1.amazonaws.com"
                            sh "docker push 589295909756.dkr.ecr.eu-west-1.amazonaws.com/esagirov-aws-bb/${it}:${BUILD_NUMBER}"
                        }
                    }
                }
            }
        }
        stage('Set tag') {
            steps {
                script {
                    withCredentials([gitUsernamePassword(credentialsId: 'github_creds', gitToolName: 'git-tool')]) {
                        sh "git tag -a ${BUILD_NUMBER} -m 'Tag ${BUILD_NUMBER}'"
                        sh "git push --tags"
                    }
                }
            }
        }
        stage('Deploy to dev') {
            steps {
                script {
                    dir('./argo') {
                        withCredentials([gitUsernamePassword(credentialsId: 'github_creds', gitToolName: 'git-tool')]) {
                            sh "git clone https://github.com/eldarsagirovda/aws-bb-containerization-argo.git ."
                            valuesFile = readYaml (file: 'envs/dev/values.yaml')
                            valuesFile.spec.source.targetRevision = "${BUILD_NUMBER}"
                            valuesFile.spec.image.tag = "${BUILD_NUMBER}"
                            writeYaml file: 'envs/dev/values.yaml', data: valuesFile, overwrite: true
                            sh """
                                git add envs/dev/values.yaml
                                git commit -m \"Update dev app version to ${BUILD_NUMBER}\"
                                git push
                            """
                        }
                        
                    }
                }
            }
        }

    }
}