pipeline {
    agent any
    environment {
        PROJECTPATH = '/jenkins/workspace/EKS-Stage-ValleSaude-OpenEHR'
        ENVIRONMENT = 'prod'
        IMAGE_TAG = sh(returnStdout: true, script: 'git rev-parse --short HEAD')
    }
    stages {
        stage('Build Application') {
            environment {
                JAVA_HOME = '/usr/java/jdk1.8.0_181/'
                PATH = '/usr/java/jdk1.8.0_181/bin:/sbin:/usr/sbin:/bin:/usr/bin'
                CLASSPATH = '.'
                CMDGRAILS = '/usr/local/grails-2.5.6/bin/grails'
                CMDCLEANALL = 'clean-all'
                CMDPRODWAR = 'prod war target/ROOT.war'
            }
            steps {
                sh "cd ${PROJECTPATH}"
                sh "${CMDGRAILS} ${CMDCLEANALL}"
                sh "${CMDGRAILS} ${CMDPRODWAR}"
            }
        }
        stage('Build & Push Docker Image') {
            environment {
                APP = "openehrserver"
            }
            steps {
                sh "mkdir -p ${PROJECTPATH}/build-deploy/${APP}"
                sh "mv ${PROJECTPATH}/target/ROOT.war ${PROJECTPATH}/build-deploy/${APP}/"
                sh "aws s3 cp s3://vallesaude-dockerfiles/release-image-jenkins.sh build-deploy/."
                sh "aws s3 cp s3://vallesaude-dockerfiles/${APP} build-deploy/${APP}/. --recursive"
                sh "chmod +x ./build-deploy/release-image-jenkins.sh"
                sh "cd build-deploy ; ./release-image-jenkins.sh ${APP} ${ENVIRONMENT}"
            }
        }
        stage('Deploy') {
            environment {
                CHART_NAME = "openehr-server"
                HELM_NAMESPACE = "prod-limited"
                APP_NAMESPACE = "prod"
                HELM_TIMEOUT = '300'
                HELM_UPGRADE = 'helm upgrade --install --tiller-namespace ${HELM_NAMESPACE} --namespace ${APP_NAMESPACE} --description ${IMAGE_TAG} --wait --timeout ${HELM_TIMEOUT} --kubeconfig ./build-deploy/${ENVIRONMENT}-kubeconfig -f ./build-deploy/${CHART_NAME}/values-${ENVIRONMENT}.yaml --set image.tag=${IMAGE_TAG} ${CHART_NAME} ./build-deploy/${CHART_NAME}/ --namespace ${ENVIRONMENT}'
            }
            steps {
                sh "mkdir -p ${PROJECTPATH}/build-deploy/${CHART_NAME}"
                sh "aws s3 cp s3://vallesaude-charts/${CHART_NAME} build-deploy/${CHART_NAME}/. --recursive"
                sh "aws s3 cp s3://vallesaude-kubeconfig/${ENVIRONMENT}-kubeconfig build-deploy/."
                sh "${HELM_UPGRADE}"
            }
        }
        stage('Clean') {
            steps {
                sh "rm -rf ${PROJECTPATH}/build-deploy"
            }
        }
    }
}
