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
            when { not { branch 'master' } }
            steps {
                sh 'echo "SONARQUBE in action......"'
                withSonarQubeEnv(installationName: "sonar", credentialsId: "student13token") {
                    sh """
                    ${scannerHome}/bin/sonar-scanner \
                    -Dsonar.projectKey=srudent13 \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=https://server2.jenkins-practice.tk:9443​ 
                      
                    """

                }

            }
        }

        stage('build') {
            when { branch 'master'}
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
            when { branch 'master'}
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
                            nexusArtifactUploader artifacts: [[artifactId: 'archive', classifier: 'student13', file: 'archive.tgz', type: 'tgz']], credentialsId: 'student13-nexus', groupId: 'mdt', nexusUrl: 'server2.jenkins-practice.tk:8443', nexusVersion: 'nexus2', protocol: 'https', repository: 'student13-repo', version: "${LATEST}"

                            
                        }
                        
                    
                }
                
        }
        
       
    }
}
