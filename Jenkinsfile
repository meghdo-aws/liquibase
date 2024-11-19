  pipeline {
      agent { label 'javaagent' }
      parameters {
         choice(name: 'action', defaultValue: 'update', choices: ['update', 'rollback'], description:'Select if you update or rollback release' )
         string(name: 'release', defaultValue: 'v1.0.0', description: 'Release to be deployed')
         string(name: 'rollbackRelease', visibility: { return params.action == 'rollback'}, description: 'Release to rollback')
       }
      environment {
          CHART_PATH = './helm-charts'   // Set the path to your Helm chart
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
                   sh """"
                    if [ ${params.action} == "update"]; then
                      if [ -f helm-charts/changelog/releases/${params.release}/*.xml]; then
                        gcloud config set project ${PROJECT_ID}
                        gcloud container clusters get-credentials ${CLUSTER} --zone ${REGION}

                        helm install liquibase-update-${params.release} ${CHART_PATH} --namespace ${NAMESPACE} --set release=${params.release} --set rollbackrelease=${params.release} --set action="update"

                        helm install liquibase-tag-${params.release} ${CHART_PATH} --namespace ${NAMESPACE} --set release=${params.release} --set rollbackrelease=${params.release} --set action="tag"
                      else
                        echo "Release changeset files not found"
                        exit 1
                      fi
                    else
                      gcloud config set project ${PROJECT_ID}
                      gcloud container clusters get-credentials ${CLUSTER} --zone ${REGION}

                      helm install liquibase-rollback-${params.release} ${CHART_PATH} --namespace ${NAMESPACE} --set release=${params.release} --set rollbackrelease=${params.rollbackRelease} --set action="rollback"
                    fi
                  """
                }
                }
          }
      }
      post {
          always {
              script {
                  cleanWs()
              }
          }
      }
  }


