pipeline {
    agent { label 'javaagent' }
    parameters {
       choice(name: 'action', choices: ['update', 'rollback'], description:'Select if you update or rollback release' )
       string(name: 'release', defaultValue: 'v1.0.0', description: 'Release to be deployed')
       string(name: 'rollbackRelease', defaultValue: 'v1.0.0', description: 'Release to rollback')
    }
    environment {
        CHART_PATH = './helm-charts'
        BASE_PATH = '/home/jenkins/agent/workspace'
        NAMESPACE = 'liquibase'
        PROJECT_ID = 'meghdo-4567'
        CLUSTER = 'meghdo-cluster'
        REGION = 'europe-west1'

        TIMESTAMP = sh(
                    script: 'date +%Y%m%d-%H%M%S',
                    returnStdout: true
                ).trim()

        JOB_IDENTIFIER = "${env.JOB_NAME}-${TIMESTAMP}"
    }
    stages {
        stage('Deploy Liquibase changes') {
            steps {
                script {
                    sh '''
                    set +x  # Exit immediately if a command exits with a non-zero status

                    if [ ${params.action} == "update" ]; then
                        # Check if changelog files exist with globbing
                        if compgen -G "helm-charts/changelog/releases/${params.release}/*.xml" > /dev/null; then
                            gcloud config set project ${PROJECT_ID}
                            gcloud container clusters get-credentials ${CLUSTER} --zone ${REGION}

                            # Update deployment
                            helm install ${JOB_IDENTIFIER} ${CHART_PATH} \
                                --namespace ${NAMESPACE} \
                                --set release=${params.release} \
                                --set rollbackrelease=${params.release} \
                                --set action="update"
                                --set jobidentifier=${JOB_IDENTIFIER}

                            # Tag deployment
                            helm install ${JOB_IDENTIFIER} ${CHART_PATH} \
                                --namespace ${NAMESPACE} \
                                --set release=${params.release} \
                                --set rollbackrelease=${params.release} \
                                --set action="tag"
                                --set jobidentifier=${JOB_IDENTIFIER}
                        else
                            echo "Release changeset files not found for ${params.release}"
                            exit 1
                        fi
                    else
                        # Rollback deployment
                        gcloud config set project ${PROJECT_ID}
                        gcloud container clusters get-credentials ${CLUSTER} --zone ${REGION}

                        helm install --set jobidentifier=${JOB_IDENTIFIER} ${CHART_PATH} \
                            --namespace ${NAMESPACE} \
                            --set release=${params.release} \
                            --set rollbackrelease=${params.rollbackRelease} \
                            --set action="rollback"
                            --set jobidentifier=${JOB_IDENTIFIER}
                    fi
                    '''
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}