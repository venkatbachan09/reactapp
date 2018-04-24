#!/usr/bin/env groovy
@Library('cplib') _



    


    pipeline {

 

        agent any

        options {
            disableConcurrentBuilds()
            buildDiscarder(logRotator(numToKeepStr: '30', daysToKeepStr: '15'))
            skipDefaultCheckout(true)
            timestamps()
            skipStagesAfterUnstable()
        }

        parameters {
            string (
                name : 'RELEASE_VERSION',
                defaultValue: '',
                description: '[Optional] Application Docker release tag (e.g. master, 1.0.0, 1.0.0.15). Please use this if you need to deploy an earlier version of the code.'
            )
            string (
                name : 'CHANGE_TICKET',
                defaultValue: '',
                description: '[N/A for Non Production Deploy] Change ticket for production deployment (e.g. C170100035).'
            )
            choice (
                name: 'PIPELINE',
                choices: ' \na\nb\nc',
                description: '[Optional] Select pipeline for anything other than the first pipeline.'
            )
        }

        stages {

            stage('Prepare') {
            	steps {
            		script {
            			log.info "Explicit scm checkout ..."
            			checkout scm
                        

            		}
            	}
            }//Close stage 'Prepare'

            stage('Build Artifacts') {
                steps {
                    script {
                        sh "ls -lt ${JENKINS_HOME_SLAVE}/tools/jenkins.plugins.nodejs.tools.NodeJSInstallation/"
                    	//def NODE_HOME = "${JENKINS_HOME_SLAVE}/tools/jenkins.plugins.nodejs.tools.NodeJSInstallation/Node_6.11.1"
                    	def NODE_HOME = "${JENKINS_HOME_SLAVE}/tools/jenkins.plugins.nodejs.tools.NodeJSInstallation/Node_8.9.1"
                    	
                    	
                        sh """
		    echo "Building artifacts ..."
		    export PATH=$PATH:$NODE_HOME/bin;
		    echo "Running node build using npm ..."
		    npm install -g npm@latest
		    rm -rf node_modules
		    npm install
		    npm install --save react react-dom react-scripts
		"""
log.info "Preparing Image contents ..."
    tmp_dir = "tmp"
    deployments_dir = tmp_dir
    log.info "Copy necessary build artifacts into the deployments dir ..."
    sh "mkdir -p ${deployments_dir}"
        sh "cp -pr *.json src public node_modules ${deployments_dir}"
    sh "chmod -R 777 ${tmp_dir}"
    log.info "Displaying artifacts ..."
    sh "ls -lt ${deployments_dir}"
    

                    }
                }
	        }//Close stage 'Build Artifacts'

	        stage('Build Image') {
                steps {
                    script {
                    
                    	if(!wspace.releaseVersionExists()) {
                    	    sh '''
                                echo "Release version not exist"
                            '''
                            appReleaseTag = osh.buildAppImage("reactapp", appReleaseTag)
                            sh '''
                                echo $appReleaseTag
                            '''
                        } else {
                             sh '''
                                echo "Release version exist"
                            '''
                            appReleaseTag = env.RELEASE_VERSION
                        }                        
                    }
                }
	        }//Close stage 'Build Image'
	        
	      
	        stage('Deploy Tsta') {
                steps {
                    script {
                        osh.deploy("ppx-test", "reactapp", "tsta", appReleaseTag)
                    }
                }
            }//Close stage 'Deploy Tsta'
            
            /*stage('Deploy Stga') {
                steps {
                    script {
                       osh.deploy("ppx-stage", "ppx-dashboard", "stga", appReleaseTag)
                    }
                }
            }*///Close stage 'Deploy stga'

            
            //stage('Deploy Prod') {
            //    steps {
            //        script {
            //            osh.deploy("ppx-prod", "ppx-dashboard", "prod", appReleaseTag)
            //        }
            //    }
            //}//Close stage 'Deploy prod'
            
            //TBD Deploy PROD still needs to be added


        }//Close stages

    }//Close pipeline
