## Configure sonarqube server for jenkins:

- Install SonarQube

```
login jenkins >> manage jankins >> Manage Plugins >> Available plugins >> SonarQube Scanner
```

- Server authentication token setup

```
login jenkins >> manage jankins >> Manage Jenkins >> Stores scoped to Jenkins (system) >> Global credentials (unrestricted) >> Add Credentials
            -- Kind : secret text
            -- scope: Global
            -- Secret: from sonarqube server
            -- ID  :
            -- Description :
```

- SonarQube server setup

```
login jenkins >> manage jankins >> Configure system >>  SonarQube servers
            -- Name
            -- Server URL
            -- Server authentication token
```

```
login jenkins >> Manage Jenkins >> Global Tool Configuration >> SonarQube Scanner
            -- SonarQube Scanner Name
            -- Install automatically from Install from Maven Central
```

**[sonar_scanner download](https://binaries.sonarsource.com/?prefix=Distribution/sonar-scanner-cli/)**

## Plugin Install:

- NodeJS

```
login jenkins >> Manage Jenkins >> Global Tool Configuration >> NodeJS
           -- Name
           -- Install automatically from Install from nodejs.org
           -- Version
           -- Global npm packages to install --> @angular/cli@12
```

- Maven

```
login jenkins >> Manage Jenkins >> Global Tool Configuration >> Maven
           -- Name
           -- Install automatically from Install from Apache
           -- Version
```

- Gradle

```
login jenkins >> Manage Jenkins >> Global Tool Configuration >> Gradle
            -- Name
            -- Install automatically from Install from Gradle.org
            -- Version
```

- JDK

```
login jenkins >> Manage Jenkins >> Global Tool Configuration >> JDK
            -- Name
            -- JAVA_HOME
```

## Testing all plugin:

- Test1:

```groove
pipeline {
    agent any

    tools {
        maven 'maven_3.8.4'
        jdk 'jdk1.8.0_291'
        gradle 'Gradle_6.0.1'
    }

    stages {

        stage ("tools-version") {
            steps {
                echo 'gradle version......'
                sh 'gradle -v'
                echo 'maven version.......'
                sh 'mvn -v'
                echo 'NodeJS version......'
                nodejs('node_14'){
                    sh 'ng version'
                }
                //def scannerHome = tool 'sonar_scanner_4.8'
                //withSonarQubeEnv('sonar_scanner_4.8') {
                   //sh '/opt/jenkin_home/tools/hudson.plugins.sonar.SonarRunnerInstallation/sonar_scanner_4.8/bin/sonar-scanner '
                //}
            }

        }
        stage('SonarQube_Analysis') {
            //def scannerHome = tool 'sonar_scanner_4.8'
            steps {
                script {
                    def scannerHome = tool 'sonar_scanner_4.8' ;
                    withSonarQubeEnv('bcps-sonar-server') {
                    sh '''/var/jenkins_home/tools/hudson.plugins.sonar.SonarRunnerInstallation/sonar_scanner_4.8/bin/sonar-scanner \
                        -Dsonar.projectKey=jenkins-test \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://119.148.13.156:9112 \
                        -Dsonar.login=e9a378390bd62ebf04f1bb99e39b2c1701072229'''
                    }

                }
            }
        }

    }
}
```

- Test2:

```
pipeline {
    agent any

    tools {
        maven 'maven_3.8.4'
        jdk 'jdk1.8.0_291'
        gradle 'Gradle_6.0.1'
    }
    environment {
		//	PATH="/opt/gradle/bin:$PATH"
			WORKSPACE= "/var/jenkins_home/workspace/BGB_QA_Deployment/source"
			//TEST_WK="/home/jenkins/workspace/BGB_QA_Deployment/source/service/hims-service/src/test/java/com/egeneration/hims/service"
			}
    stages {

        //stage('Preparation') {
		    //steps {
			    //git branch: 'development', credentialsId: 'egengit', url: 'https://gitlab.com/healtix.project/bg-hmis.git'
		    //}
        //}

        stage('Build_Gateway') {
			steps {
			    echo 'gradle version......'
                sh 'gradle -v'
                echo 'maven version.......'
                sh 'mvn -v'
                sh ''' cd $WORKSPACE/service/hims-api-gateway && gradle sonarqube \
                        -Dsonar.projectKey=bgb-qa-gateway \
                        -Dsonar.host.url=http://119.148.13.156:9112 \
                        -Dsonar.login=94779b281f0135aa2acd88cead65d94935c0bd22'''
				//sh 'cd $WORKSPACE/service/hims-api-gateway && gradle clean build'
			}
		}

		//stage('Build_Client') {
			//environment {
					//NODE_OPTIONS="--max-old-space-size=4096"
			//}
			//steps {
			    //nodejs('node_14'){
			        //sh 'cd $WORKSPACE/client/hims-client && npm install && ng build --aot --prod --base-href /hmis-client/ && mv dist hmis-client && zip -r hmis-client.zip hmis-client/'
			    //}

			//}
		//}

    }
}
```

- Test3:

```
pipeline {
    agent any
    tools{
        maven 'maven_3.8.4'
        jdk 'jdk1.8.0_291'
        gradle 'Gradle_6.0.1'
    }
    environment {
        WDIR= ""
        srvIP = ""
    }
    stages{
        stage('git') {
            steps {
                environment {}
                git branch: '', credentialsId: '', url: ''
            }
        }
        stage('SonarQube_Analysis') {
            environment {}
            steps {
                script {
                    def scannerHome = tool 'tools_name' ;
                    withSonarQubeEnv('server_name') {
                        sh ''' '''
                    }
                }
            }
        }
        stage('') {
            environment {}
            steps {}
        }
    }
}
```

- Test4:

```
pipeline {
    agent any
    tools{
        maven 'maven_3.8.4'
        jdk 'jdk1.8.0_291'
        gradle 'Gradle_6.0.1'
    }
    environment {
        WDIR= "/var/jenkins_home/workspace/lms-service/lms/"
    }
    stages{
        stage('git') {
            steps {
                git branch: 'test', credentialsId: 'egengit', url: 'https://gitlab.com/egenlms/lmscore.git'
            }
        }
        stage('SonarQube_Analysis') {
            steps {
                withSonarQubeEnv('bcps-sonar-server') {
                    sh '''cd $WDIR && mvn sonar:sonar \
                     -Dsonar.projectKey=lmscore \
                     -Dsonar.host.url=http://119.148.13.156:9112 \
                     -Dsonar.login=b39cb0db8a6669194cbe5082f663c5679560f86f '''
                }
            }
        }
        stage('Build_lms_web') {
			steps {
				sh 'cd $WDIR && mvn clean install -Ptest'
			}
		}
    }
}
```

```
pipeline {
	agent any
	tools {
        maven 'Maven_3.6.3'
        jdk 'jdk1.8.0_291'
        gradle 'Gradle_6.0.1'
    }
	environment {
		//	PATH="/opt/gradle/bin:$PATH"
			WORKSPACE= "/home/jenkins/workspace/BGB_QA_Deployment/source"
			TEST_WK="/home/jenkins/workspace/BGB_QA_Deployment/source/service/hims-service/src/test/java/com/egeneration/hims/service"
			}

	stages {
		stage('Preparation') {
		    steps {
			    git branch: 'development', credentialsId: 'Git_User_Jenkins', url: 'https://gitlab.com/healtix.project/bg-hmis.git'
				sh 'cd $WORKSPACE/client/hims-client && rm -fr hmis-client hmis-client.zip'
			}
		}

		stage('Build_Client') {
			environment {
					NODE_OPTIONS="--max-old-space-size=4096"
			}
			steps {
				sh 'cd $WORKSPACE/client/hims-client && npm install && ng build --aot --prod --base-href /hmis-client/ && mv dist hmis-client && zip -r hmis-client.zip hmis-client/'
			}
		}
		// stage('Build_Gateway') {
		//	steps {
		//		sh 'cd $WORKSPACE/service/hims-api-gateway && gradle clean build'
		//	}
		//}
		stage('Build_Service') {
			steps {
				sh 'rm -fr $TEST_WK/HimsServiceApplicationTests.java && cp /home/cmsd_vm/HimsServiceApplicationTests.java $TEST_WK && cd $WORKSPACE/service/hims-service && gradle clean build'
			}
		}
	    stage('Deployed_222') {
	        environment {
					WS= "/bgb/test_server/apache-tomcat-9.0.30/webapps"
			}
			steps {
				sshagent(['SSH_USER_222']) {
					sh '''ssh -o StrictHostKeyChecking=no egen@116.68.194.222 /bgb/test_server/apache-tomcat-9.0.30/bin/catalina.sh stop
                    ssh -o StrictHostKeyChecking=no egen@116.68.194.222 rm -fr $WS/hims-service $WS/hims-service.war $WS/hmis-client $WS/hmis-client.zip
                    scp -o StrictHostKeyChecking=no $WORKSPACE/client/hims-client/hmis-client.zip egen@116.68.194.222:$WS/
                    scp -o StrictHostKeyChecking=no $WORKSPACE/service/hims-service/build/libs/hims-service.war egen@116.68.194.222:$WS/
                    ssh -o StrictHostKeyChecking=no egen@116.68.194.222 unzip $WS/hmis-client.zip -d $WS/
                    ssh -o StrictHostKeyChecking=no egen@116.68.194.222 /bgb/test_server/apache-tomcat-9.0.30/bin/catalina.sh start'''
				}
			}
		}
	}
	post {
       failure {
                emailext attachLog: true,
                body: "${currentBuild.currentResult}: Job ${env.JOB_NAME}:${env.BUILD_NUMBER}\nMore Info can be found here: ${env.BUILD_URL}",
                compressLog: true,
                recipientProviders: [[$class: 'DevelopersRecipientProvider'],
                [$class: 'RequesterRecipientProvider']],
                    replyTo: 'do-not-reply@company.com',
                    subject: "Status: ${currentBuild.result?:'Failed'} - Job \'${env.JOB_NAME}:${env.BUILD_NUMBER}\'",
                     to: 'faria.sultana@egeneration.co rashed.zaman@egeneration.co mohammad.shahajada@egeneration.co nadir.chowdhury@generation.co sohana.khondodoker@generation.co'
        }
        success{
            emailext attachLog: true,
            body: "${currentBuild.currentResult}: Job ${env.JOB_NAME}:${env.BUILD_NUMBER}\nMore Info can be found here: ${env.BUILD_URL}",
                compressLog: true,
                recipientProviders: [[$class: 'DevelopersRecipientProvider'],
                [$class: 'RequesterRecipientProvider']],
                    replyTo: 'do-not-reply@company.com',
                    subject: "Status: ${currentBuild.result?:'Success'} - Job \'${env.JOB_NAME}:${env.BUILD_NUMBER}\'",
                     to: 'faria.sultana@egeneration.co rashed.zaman@egeneration.co mohammad.shahajada@egeneration.co nadir.chowdhury@generation.co sohana.khondodoker@generation.co'

        }
    }
}
```
