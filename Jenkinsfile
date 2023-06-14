pipeline {
    agent {
        label "sys-admin-mnf"
    }
    parameters {
        choice(name: 'ENV_ITI', choices: ['test', 'build'])
    
    stages {
        stage('build') {
            steps {
                script {
                    echo 'build'
                    if (params.ENV_ITI == "build") {
                        withCredentials([usernamePassword(credentialsId: 'iti-sys-admin-mnf-docker-cred', usernameVariable: 'USERNAME_SYSADMIN', passwordVariable: 'PASSWORD_SYSADMIN')]) {
                            sh """
                                docker login -u ${USERNAME_SYSADMIN} -p ${PASSWORD_SYSADMIN}
                                docker build -t mohamedtayef/bakehouseitisysadmin:v${BUILD_NUMBER} .
                                docker push mohamedtayef/bakehouseitisysadmin:v${BUILD_NUMBER}
                                echo ${BUILD_NUMBER} > ../build_num.txt
                                echo ${ENV_ITI}
                            """
                        }
                    } else {
                        echo "user chose ${params.ENV_ITI}"
                    }
                }
            }
        }
        stage('deploy') {
            steps {
                echo 'deploy'
                script {
                    if (params.ENV_ITI == "test") {
                        withCredentials([file(credentialsId: 'iti-sys-admin-mnf-Kubeconfig-cred', variable: 'KUBECONFIG_ITI')]) {
                            sh """
                                export BUILD_NUMBER=$(cat ../build_num.txt)
                                mv Deployment/deploy.yaml Deployment/deploy.yaml.tmp
                                cat Deployment/deploy.yaml.tmp | envsubst > Deployment/deploy.yaml
                                rm -rf Deployment/deploy.yaml.tmp
                                kubectl apply -f Deployment --kubeconfig ${KUBECONFIG_ITI} -n ${ENV_ITI}
                            """
                        }
                    } else {
                        echo "user chose ${params.ENV_ITI}"
                    }
                }
            }
        }
    }
}
