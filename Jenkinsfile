pipeline{
    agent any
    tools {
    maven 'Maven'
    }
    stages{
        stage('Git-Checkout'){
            steps{
                echo "Checking out from Git Repo";
                git 'https://github.com/mark-cabardo/webapp.git'
            }
        }
        
        stage('Build'){
            steps{
                echo "Building out the checked-out project";
                      sh "mvn clean install package -DskipTests"
            }
        }
		
		stage("Unit Test"){
            steps{
                echo 'This job runs unit tests on Java Spring reference application.'
                sh "mvn test"
            }  
        }
		
		stage("Code Analysis"){
            steps{
                echo 'This job runs SonarQube code quality testing.'
                sh "mvn sonar:sonar"
            }  
        }
        
        stage('Deploy-SSH-Copy'){
            steps{
                echo "Copying war file from Jenkins";
                sshPublisher(publishers: [sshPublisherDesc(configName: 'docker_host', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '//opt//docker', remoteDirectorySDF: false, removePrefix: 'target', sourceFiles: 'target/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
        }
        
        stage('Deploy-Push image'){
            steps{
                echo "Pushing image to dockerhub account";
                sshPublisher(publishers: [sshPublisherDesc(configName: 'docker_host', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''cd /opt/docker
docker build -t $JOB_NAME:v1.$BUILD_ID .
docker tag $JOB_NAME:v1.$BUILD_ID markymark08/$JOB_NAME:v1.$BUILD_ID
docker tag $JOB_NAME:v1.$BUILD_ID markymark08/$JOB_NAME:latest
docker push markymark08/$JOB_NAME:v1.$BUILD_ID
docker push markymark08/$JOB_NAME:latest
docker rmi $JOB_NAME:v1.$BUILD_ID markymark08/$JOB_NAME:v1.$BUILD_ID markymark08/$JOB_NAME:latest''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '//opt//docker', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'Dockerfile')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
        }
        
        stage('Deploy-Pull image'){
            steps{
                echo "Pulling image from docker hub";
                sshPublisher(publishers: [sshPublisherDesc(configName: 'ansible_server', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''cd /opt/playbooks
ansible-playbook -i /opt/playbooks/hosts create_docker_container_demo.yml''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
        }
        
    }
    
    post{
        always{
            echo "This will always run"
	    archiveArtifacts "target/**/*"
            junit 'target/surefire-reports/*.xml'
	    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'coverage', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '']) 
        }
        success{
            echo "This will run only if successful"
        }
        failure{
            echo "This will run only if failure"
        }
        unstable{
            echo "This will run only if unstable"
        }
        changed{
            echo "This will run only if the state of the pipeline has changed"
            echo "For example, if the pipeline was previously failing but is now successful"
        }
    }
}
