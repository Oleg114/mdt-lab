pipeline {
    agent {
        label 'student13'
    }
    
    stages {
        stage('checkout') {
            steps {
                checkout scm
                sh 'echo "-------------"'
                sh "echo $BRANCH_NAME"
                sh 'echo "-------------"'
            }
        }
        
        stage('PR- sonar scanner') {
            when {
                 expression {return BRANCH_NAME != 'master';}
                 }
            steps {
                sh 'echo "SONARQUBE in action......"'
                withSonarQubeEnv(installationName: "sonarqube-default", credentialsId: "student13token") {
                    script {
                    def sonarHome = tool 'sonarqube-scanner-4.0'
                    sh """${sonarHome}/bin/sonar-scanner -Dsonar.projectKey=student13-project -Dsonar.sources=www"""
                    }

                }

            }
        }
        stage('Quality gate') {
            when {
                 expression {return BRANCH_NAME != 'master';}
                 }
            steps {
                waitForQualityGate abortPipeline: true
            }
        }

        stage('build') {
            tools {
                nodejs 'Node12'
             }
            when {
                 expression {return BRANCH_NAME == 'master';} 
                 }
                steps {
                    
                    script {
                        sh  """
                        echo "BUILD......"
                        cd $WORKSPACE/www/css
                        cleancss -d style.css > ../min/custom-min.css
                        cd $WORKSPACE/www/js
                        uglifyjs --timings init.js -o ../min/custom-min.js
                        """
                    }
                }
        }        
        stage('archive'){
            when {
                 expression {return BRANCH_NAME == 'master';}
                 }
                steps {
                        script {
                            if (fileExists('../../version.txt')) {
                            echo 'version.txt Exist'
                            VERSION=sh(returnStdout: true, script: 'sed -n 1p ../../version.txt').trim()
                            } 
                            else {
                            VERSION='1.0.0'
                            }
                            LATEST="${VERSION}-${BUILD_ID}"
                            sh  """
                            echo ${LATEST}
                            echo ${LATEST} >../../latest.txt
                            
                            cd ${WORKSPACE}/www
                            tar --exclude=\"./css\"--exclude=\"./js\" -c -z -f "../archive.tgz" .
                                
                            """
                            nexusArtifactUploader artifacts: [[artifactId: 'archive', classifier: '', file: 'archive.tgz', type: 'tgz']], credentialsId: 'student13-jenkins-nexus', groupId: 'mdt', nexusUrl: 'server2.jenkins-practice.tk:8443', nexusVersion: 'nexus2', protocol: 'https', repository: 'student13-repo', version: "${LATEST}"
                            
                            
                        }
                        
                    
                }
                
        }
        
       
    }
}
