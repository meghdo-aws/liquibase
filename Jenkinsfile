pipeline {
    agent {
            kubernetes {
                label 'meghdo-java'
                yamlFile "pipeline/pod.yaml"
            }
    }
    parameters {
       choice(name: 'action', choices: ['update', 'rollback'], description:'Select if you update or rollback release' )
       string(name: 'release', defaultValue: 'v1.0.0', description: 'Release to be deployed')
       string(name: 'rollbackRelease', defaultValue: 'v1.0.0', description: 'Release to rollback')
       booleanParam(name: 'firstrun', defaultValue: false, description: 'Is this the first run?')
    }
    environment {
        CHART_PATH = './helm-charts'
        BASE_PATH = '/home/jenkins/agent/workspace'
        NAMESPACE = 'liquibase'
        SERVICEACCOUNT = 'liquibase'
        PROJECT_ID = 'meghdo-4567'
        CLUSTER = 'meghdo-cluster'
        REGION = 'europe-west1'
    }
    stages {
        stage('Deploy Liquibase changes') {
            steps {
                script {
                    container('infra-tools') {
                    // Generate TIMESTAMP and JOB_IDENTIFIER dynamically within the script block
                    def timestamp = sh(script: "date +%s", returnStdout: true).trim()
                    def jobIdentifier = "${env.JOB_NAME}-${timestamp}".replaceAll(/[^a-zA-Z0-9.-]/, "-").toLowerCase()
                    sh """
                    if [ "${params.action}" = "update" ]; then
                        if find "helm-charts/changelog/releases/${params.release}" -maxdepth 1 -name "*.xml" | grep -q .; then
                            gcloud config set project ${PROJECT_ID}
                            gcloud iam service-accounts add-iam-policy-binding svc-gke@${PROJECT_ID}.iam.gserviceaccount.com --member="serviceAccount:${PROJECT_ID}.svc.id.goog[${NAMESPACE}/${SERVICEACCOUNT}]" --role="roles/iam.workloadIdentityUser" 
                            gcloud container clusters get-credentials ${CLUSTER} --zone ${REGION}

                            # Update deployment
                            helm install update-${jobIdentifier} ${CHART_PATH} \
                                --create-namespace \
                                --namespace ${NAMESPACE} \
                                --set release=${params.release} \
                                --set rollbackrelease=${params.release} \
                                --set action="update" \
                                --set jobidentifier="update-${jobIdentifier}" 

                            if [ "${params.firstrun}" = "true" ]; then
                             sleep 180
                            fi
                            # Tag deployment
                            helm install tag-${jobIdentifier} ${CHART_PATH} \
                                --create-namespace \
                                --namespace ${NAMESPACE} \
                                --set release=${params.release} \
                                --set rollbackrelease=${params.release} \
                                --set action="tag" \
                                --set jobidentifier="tag-${jobIdentifier}" 
                        else
                            echo "Release changeset files not found for ${params.release}"
                            exit 1
                        fi
                    else
                        # Rollback deployment
                        gcloud config set project ${PROJECT_ID}
                        gcloud container clusters get-credentials ${CLUSTER} --zone ${REGION}

                        helm install rollback-${jobIdentifier} ${CHART_PATH} \
                            --create-namespace \
                            --namespace ${NAMESPACE} \
                            --set release=${params.release} \
                            --set rollbackrelease=${params.rollbackRelease} \
                            --set action="rollback" \
                            --set jobidentifier="rollback-${jobIdentifier}" 
                    fi
                    """

                    }     
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
