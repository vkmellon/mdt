pipeline {
    agent {
        label 'vitalii_hirenko'
    }
    parameters {
        choice choices: ['DEVELOP', 'RELEASE'], description: '', name: 'RELEASE'
        string defaultValue: '0.0.1', description: '', name: 'RELEASE_VER', trim: false
    }

    tools {
        nodejs 'Node12'
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/dbielik/mdt'
            }
        }
        // stage('Scan Sonar') {
        //     steps {
        //         withSonarQubeEnv(installationName: 'sonarqube-external', credentialsId: 'sonarqube-server') {
        //             script {
        //                 sonarHome = tool 'sonarscanner4'
        //                 sh """
        //                 ${sonarHome}/bin/sonar-scanner -Dsonar.projectKey=www -Dsonar.sources=www
        //                 """
        //             }
        //         }
        //     }
            
        // }
        // stage('Quality gate') {
        //     steps {
        //         waitForQualityGate abortPipeline: true
        //     }
        // }
        stage('Build') {
            parallel {
                stage('JS') {
                    steps {
                        sh label: 'minimize JS', script: """
                        cd ${WORKSPACE}/www/js
                        uglifyjs --timings init.js -o ../min/custom-min.js
                        """
                    }
                }
                stage('CSS') {
                    steps {
                        sh label: 'minimize CSS', script: """
                        cd ${WORKSPACE}/www/css
                        cleancss -d style.css > ../min/custom-min.css"""
                    }
                }
                stage('Prepare artifact') {
                    steps {
                        sh label: 'archive', script: """
                        cd ${WORKSPACE}/www
                        tar --exclude='./css' --exclude='./js' -c -z -f ../site-archive-${params.RELEASE}-${params.RELEASE_VER}-${BUILD_NUMBER}.tgz ."""
                    }
                }
            }
        }
        stage('Archive') {
            when {
                expression {
                    params.RELEASE == 'RELEASE'
                }
            }
            steps {
                archiveArtifacts '*.tgz'
            }
        }
        stage('UploadArtifact') {
            when {
                expression {
                    params.RELEASE == 'RELEASE'
                }
            }
            steps {
                script {
                    nexusArtifactUploader artifacts: [[artifactId: 'site-archive', \
                     classifier: '', \
                     file: "site-archive-${params.RELEASE}-${params.RELEASE_VER}-${BUILD_NUMBER}.tgz", \
                     type: 'tgz']], \
                     credentialsId: 'vhirenko-nexus', \
                     groupId: 'site-archive', \
                     nexusUrl: 'master.jenkins-practice.tk:9443', \
                     nexusVersion: 'nexus3', \
                     protocol: 'https', \
                     repository: 'raw-demo-hosted', \
                     version: '${RELEASE_VER}-${BUILD_NUMBER}-hirenkovitalii'
                }
            }
        }
    }
}
