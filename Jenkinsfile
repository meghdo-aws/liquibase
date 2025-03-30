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
        NAMESPACE = 'default'
        SERVICEACCOUNT = 'liquibase'
        CLUSTER = 'meghdo-cluster'
        REGION = 'us-east-1'
    }
    stages {
        stage('Connect to EKS Cluster') {
            steps {
                script {
                    // Use AWS container to connect to EKS cluster
                    container('aws') {
                        sh """
                        aws eks update-kubeconfig --name ${CLUSTER} --region ${REGION}
                        """
                    }
                }
            }
        }

        stage('Deploy Liquibase changes') {
            steps {
                script {
                    // Generate TIMESTAMP and JOB_IDENTIFIER dynamically
                    def timestamp = sh(script: "date +%s", returnStdout: true).trim()
                    def jobIdentifier = "${env.JOB_NAME}-${timestamp}".replaceAll(/[^a-zA-Z0-9.-]/, "-").toLowerCase()

                    // Use infra-tools container for Helm commands
                    container('infra-tools') {
                        sh """
                        if [ "${params.action}" = "update" ]; then
                            if find "helm-charts/changelog/releases/${params.release}" -maxdepth 1 -name "*.xml" | grep -q .; then
                                # Update deployment
                                helm install update-${jobIdentifier} ${CHART_PATH} \\
                                    --create-namespace \\
                                    --namespace ${NAMESPACE} \\
                                    --set release=${params.release} \\
                                    --set rollbackrelease=${params.release} \\
                                    --set action="update" \\
                                    --set jobidentifier="update-${jobIdentifier}"

                                if [ "${params.firstrun}" = "true" ]; then
                                 sleep 180
                                fi
                                # Tag deployment
                                helm install tag-${jobIdentifier} ${CHART_PATH} \\
                                    --create-namespace \\
                                    --namespace ${NAMESPACE} \\
                                    --set release=${params.release} \\
                                    --set rollbackrelease=${params.release} \\
                                    --set action="tag" \\
                                    --set jobidentifier="tag-${jobIdentifier}"
                            else
                                echo "Release changeset files not found for ${params.release}"
                                exit 1
                            fi
                        else
                            # Rollback deployment
                            helm install rollback-${jobIdentifier} ${CHART_PATH} \\
                                --create-namespace \\
                                --namespace ${NAMESPACE} \\
                                --set release=${params.release} \\
                                --set rollbackrelease=${params.rollbackRelease} \\
                                --set action="rollback" \\
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