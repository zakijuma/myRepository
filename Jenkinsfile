    pipeline {
        agent any

        tools {
            maven 'Maven 3.6.2'//3.5.3
            jdk 'JDK-8u131'
        }

        options {
            buildDiscarder(logRotator(numToKeepStr: '2'))
        }
		
		//Deze waarden kunnen met een writeMavenPom in de pom gezet worden voordat er gebuild wordt
        parameters {
            booleanParam(name: 'RELEASE', defaultValue: false, description: 'Is this a release?')
            string(name: 'werkgeversFrontEnd', defaultValue: 'x.y.z-SNAPSHOT', description: 'Input Werkgever frontend version.')
			string(name: 'werkgeversBackEnd', defaultValue: 'x.y.z-SNAPSHOT', description: 'Input Werkgever backend version.')
        }

        stages {
            stage('Prepare workspace') {
                  steps {
                    script {
                        if (params.RELEASE) {
						echo "Dit is een Release"
                            deleteDir()
                        } else {
							echo "Dit is geen Release"
                            bat "mvn clean"
                        }
                    }
                }
            }
			stage('Checkout source code ') {
                steps {
                    //bat 'printenv'
                    script {
                        checkout scm
                    }
                }
            }
			stage('Compile and run unit test') {
                steps {
                    script {
                        if (params.ENVIRONMENT == 'NotStaging') {

                            bat "mvn package"

                            try {
                                jacoco(
                                        execPattern: '**/target/*.exec',
                                        classPattern: '**/target/classes',
                                        sourcePattern: '**/src/main/java',
                                        exclusionPattern: '**/src/test*'
                                )

                                junit '**/target/surefire-reports/TEST-*.xml'

                            } catch (ignore) {
                                // Data collection will fail when no surefire-reports are available. (E.g. there are no unit tests)
                            }

                        } else if (params.ENVIRONMENT != 'NotStaging' &&
                                (env.BRANCH_NAME =~ /RB_.*/ ||
                                        env.BRANCH_NAME =~ /master/)) {

                            bat "mvn clean package -Dmaven.test.skip=true"
                        }
                    }
                }
            }
			stage('jacoco Report') {
                steps {
					script {
					bat "mvn package"
                    try {
                        jacoco(
                            execPattern: '**/target/*.exec',
                            classPattern: '**/target/classes',
                            sourcePattern: '**/src/main/java',
                            exclusionPattern: '**/src/test*'
                             )

                        junit '**/target/surefire-reports/TEST-*.xml'

                    } catch (ignore) {
                            // Data collection will fail when no surefire-reports are available. (E.g. there are no unit tests)
                        }
					}
                }
            }
			stage('Get version and name from POM'){
                steps{
                script {
					//def pomArtifactId = bat returnStdout: true ,script: 'cd werkgevers && mvn help:evaluate -Dexpression=project.artifactId -q -DforceStdout'
                    //def pomVersion = bat returnStdout: true ,script: 'cd werkgevers && mvn help:evaluate -Dexpression=project.version -q -DforceStdout'
				
					def pomArtifactId = readMavenPom().getArtifactId()
                    def pomVersion = readMavenPom().getVersion()
					echo 'Pom values in definitionstage: '
                    echo "pomArtifactId"
					echo pomArtifactId
                    echo "pomVersion"
					echo pomVersion

                    def workspacepath = "${WORKSPACE}"
                    echo 'Workspace path is: '
                    echo workspacepath
                    def workspacepathenv = "${env.WORKSPACE}"
                    echo 'Workspace path with env in var is: '
                    echo workspacepathenv

                    def darPathTest = "${env.WORKSPACE}" + "/target/" + pomArtifactId + "-" + pomVersion + ".dar"
                    echo 'darPathTest '
                    echo darPathTest

                }
                }
            }
        }
    }