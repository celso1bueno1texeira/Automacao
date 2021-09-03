#!/usr/bin/env groovy
pipeline {
    agent { node { label 'worker' } }

    parameters {
        choice(name: 'SQUAD', choices: ['rpa', 'erp-prestadores', 'devops'])
        string(name: 'NOMEPROJETO', description: 'ATENÇÃO! \n- não pode conter caracteres especiais \n- não pode conter numeros \n- pode conter apenas "traço" (-) \n- tem que ser tudo minuscutlo \n exemplo: repo-dahora \n é o nome do repo caso ja exista')
        choice(name: 'CRIAREPOGIT', choices: ['yes', 'no'], description: 'Criar repositorio git?')
        choice(name: 'TEMPLATE', choices: ['nodejs', 'python3', 'vazio'], description: 'Linguagem do repositorio')
        choice(name: 'CRIARECR', choices: ['yes', 'no'], description: 'Criar repositorio ECR?')
        choice(name: 'CRIARGITOPS', choices: ['yes', 'no'], description: 'Criar GITOPS?')
        choice(name: 'CRIARPIPELINE', choices: ['yes', 'no'], description: 'Criar Pipeline Jenkins?')
    }
    options {
        disableConcurrentBuilds()
    }

    stages {

        stage('GIT'){
            when { expression { params.CRIAREPOGIT == "yes" } }
            steps{
                script {
                    withCredentials([usernameColonPassword(credentialsId: 'bitbucket', variable: 'TOKENBITBUCKET')]) {
                        sh("./02-create-repository/index.sh $TOKENBITBUCKET '$SQUAD-$NOMEPROJETO' $TEMPLATE ")
                    }
                }
            }
        }


        stage('ECR'){
            when { expression { params.CRIARECR == "yes" } }
            steps{
                script {
                    withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh("./01-create-registry/index.sh '$SQUAD-$NOMEPROJETO'")
                    }
                }
            }
        }

        stage('GITOPS-FILES'){
            when { expression { params.CRIARGITOPS == "yes" } }
            steps{
                script {
                    withCredentials([usernameColonPassword(credentialsId: 'bitbucket', variable: 'TOKENBITBUCKET')]){
                        sh("./03-create-gitops-files/index.sh $TOKENBITBUCKET '$SQUAD-$NOMEPROJETO'")
                    }
                }
            }
        }

        stage('GITOPS-APP-ARGO'){
            when { expression { params.CRIARGITOPS == "yes" } }
            steps{
                 script {
                    withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-prd', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh("./04-create-gitops-app-argo/index.sh $NOMEPROJETO")
                    }
                }
            }
        }

        // stage('JENKINSFILE'){
        //     when {
        //         expression { params.CRIARPIPELINE == 'yes' }
        //     }
        //     steps{
        //          script {
        //             withCredentials([string(credentialsId: 'bitbucket', variable: 'TOKENBITBUCKET')]) {
        //                 sh("./05-create-jenkinsfile/index.sh $TOKENBITBUCKET $NOMEPROJETO")
        //             }
        //         }
        //     }
        // }

        // stage('PIPELINE'){
        //     when {
        //         expression { params.CRIARPIPELINE == 'yes' }
        //     }
        //     steps{
        //          script {
        //             withCredentials([string(credentialsId: 'jenkins-token', variable: 'TOKENJENKINS')]) {
        //                 sh("./06-create-pipeline/index.sh $TOKENJENKINS $NOMEPROJETO")
        //             }
        //         }
        //     }
        // }

    } //end stages

    post {
        always {
            cleanWs()
        }
    }
}