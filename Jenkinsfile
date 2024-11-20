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
                    // Generate TIMESTAMP and JOB_IDENTIFIER dynamically within the script block
                    def timestamp = sh(script: "date +%s", returnStdout: true).trim()
                    def jobIdentifier = "${env.JOB_NAME}-${timestamp}"

                    sh """
                    if [ "${params.action}" = "update" ]; then
                        if find "helm-charts/changelog/releases/${params.release}" -maxdepth 1 -name "*.xml" | grep -q .; then
                            gcloud config set project ${PROJECT_ID}
                            gcloud container clusters get-credentials ${CLUSTER} --zone ${REGION}

                            # Update deployment
                            helm install ${jobIdentifier} ${CHART_PATH} \
                                --namespace ${NAMESPACE} \
                                --set release=${params.release} \
                                --set rollbackrelease=${params.release} \
                                --set action="update" \
                                --set jobidentifier=${jobIdentifier}

                            # Tag deployment
                            helm install ${jobIdentifier} ${CHART_PATH} \
                                --namespace ${NAMESPACE} \
                                --set release=${params.release} \
                                --set rollbackrelease=${params.release} \
                                --set action="tag" \
                                --set jobidentifier=${jobIdentifier}
                        else
                            echo "Release changeset files not found for ${params.release}"
                            exit 1
                        fi
                    else
                        # Rollback deployment
                        gcloud config set project ${PROJECT_ID}
                        gcloud container clusters get-credentials ${CLUSTER} --zone ${REGION}

                        helm install ${jobIdentifier} ${CHART_PATH} \
                            --namespace ${NAMESPACE} \
                            --set release=${params.release} \
                            --set rollbackrelease=${params.rollbackRelease} \
                            --set action="rollback" \
                            --set jobidentifier=${jobIdentifier}
                    fi
                    """
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