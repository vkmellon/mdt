pipeline {
    agent {
        label 'Demo'
    }
    parameters {
        choice choices: ['DEVELOP', 'RELEASE'], description: '', name: 'RELEASE'
        string defaultValue: '0.0.1', description: '', name: 'RELEASE_VER', trim: false
    }
    tools {
        nodejs 'Node12'
    }
    stages {
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
                    when {
                      branch 'master'
                    }
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
                branch 'master'
            }
            steps {
                archiveArtifacts '*.tgz'
            }
        }
    }
}
