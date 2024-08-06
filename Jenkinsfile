library identifier: 'RHTAP_Jenkins@main', retriever: modernSCM(
  [$class: 'GitSCMSource',
   remote: 'https://github.com/jkopriva/tssc-sample-jenkins.git'])


pipeline {
    agent {
        kubernetes {
            label 'jenkins-agent'
            cloud 'openshift'
            serviceAccount 'jenkins'
            podRetention onFailure()
            idleMinutes '30'
            containerTemplate {
                name 'jnlp'
                image 'image-registry.openshift-image-registry.svc:5000/jenkins/jenkins-agent-base:latest'
                ttyEnabled true
                args '${computer.jnlpmac} ${computer.name}'
            }
        }
    }
    environment {
        ROX_API_TOKEN = credentials('ROX_API_TOKEN')
        ROX_CENTRAL_ENDPOINT = credentials('ROX_CENTRAL_ENDPOINT')
        GITOPS_AUTH_PASSWORD = credentials('GITOPS_AUTH_PASSWORD')
        QUAY_IO_CREDS = credentials('QUAY_IO_CREDS')
    }
    stages {
        stage('init') {
            steps {
                script {
                    rhtap.info('init')
                    rhtap.init()
                }
            }
        }
        stage('build') {
            steps {
                script {
                    rhtap.info('buildah_rhtap')
                    rhtap.buildah_rhtap()
                    rhtap.merge_sboms()
                }
            }
        }
        stage('sign-attest') {
            environment {
                COSIGN_SECRET_PASSWORD = credentials('COSIGN_SECRET_PASSWORD')
                COSIGN_SECRET_KEY = credentials('COSIGN_SECRET_KEY')
                COSIGN_PUBLIC_KEY = credentials('COSIGN_PUBLIC_KEY')
            }
            steps {
                script {
                    rhtap.info('cosign_sign_attest')
                    rhtap.cosign_sign_attest()
                }
            }
        }
        stage('scan') {
            steps {
                script {
                    rhtap.info('acs_deploy_check')
                    rhtap.acs_deploy_check()
                    rhtap.info('acs_image_check')
                    rhtap.acs_image_check()
                    rhtap.info('acs_image_scan' )
                    rhtap.acs_image_scan()
                }
            }
        }
        stage('deploy') {
            steps {
                script {
                    rhtap.info('update_deployment')
                    rhtap.update_deployment()
                }
            }
        }
        stage('summary') {
            steps {
                script {
                    rhtap.info('show_sbom_rhdh')
                    rhtap.show_sbom_rhdh()
                    rhtap.info('summary')
                    rhtap.summary()
                }
            }
        }
    }
}
