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
    }
    stages {
        stage('Deploy Liquibase changes') {
            steps {
                script {
                    // Fix 1: Use shell script with proper syntax
                    sh '''
                    set -e  # Exit immediately if a command exits with a non-zero status

                    if [ "${ACTION}" == "update" ]; then
                        # Check if changelog files exist with globbing
                        if compgen -G "helm-charts/changelog/releases/${RELEASE}/*.xml" > /dev/null; then
                            gcloud config set project ${PROJECT_ID}
                            gcloud container clusters get-credentials ${CLUSTER} --zone ${REGION}

                            # Update deployment
                            helm install liquibase-update-${RELEASE} ${CHART_PATH} \
                                --namespace ${NAMESPACE} \
                                --set release=${RELEASE} \
                                --set rollbackrelease=${RELEASE} \
                                --set action="update"

                            # Tag deployment
                            helm install liquibase-tag-${RELEASE} ${CHART_PATH} \
                                --namespace ${NAMESPACE} \
                                --set release=${RELEASE} \
                                --set rollbackrelease=${RELEASE} \
                                --set action="tag"
                        else
                            echo "Release changeset files not found for ${RELEASE}"
                            exit 1
                        fi
                    else
                        # Rollback deployment
                        gcloud config set project ${PROJECT_ID}
                        gcloud container clusters get-credentials ${CLUSTER} --zone ${REGION}

                        helm install liquibase-rollback-${ROLLBACK_RELEASE} ${CHART_PATH} \
                            --namespace ${NAMESPACE} \
                            --set release=${RELEASE} \
                            --set rollbackrelease=${ROLLBACK_RELEASE} \
                            --set action="rollback"
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