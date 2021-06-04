try{
    node{
        def mavenHome
        def mavenCMD
        def docker
        def dockerCMD
        def tagName = "1.0"
        
        stage('Preparation'){
            echo "Preparing the Jenkins environment with required tools..."
            mavenHome = tool name: 'Maven_1', type: 'maven'
            mavenCMD = "${mavenHome}/bin/mvn"
            docker = tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
            dockerCMD = "$docker/bin/docker"
        }
        
        stage('git checkout'){
            echo "Checking out the code from git repository..."
            git 'https://github.com/Aniket-Utekar/batch10.git'
        }
        
        stage('Build, Test and Package'){
            echo "Building the springboot application..."
            sh "${mavenCMD} clean package"
        }
        
        stage('Sonar Scan'){
            echo "Scanning application for vulnerabilities..."
            withSonarQubeEnv(credentialsId: 'sonar_token') {
            sh "${mavenCMD} sonar:sonar -Dsonar.host.url=http://34.72.217.230:9000"
        }
        }

        stage('Integration test'){
            echo "Executing Regression Test Suits..."
            echo "Generating Test Report"
            sh "${mavenCMD} surefire-report:report-only"
        }
        
        stage('publish report'){
            echo " Publishing HTML report.."
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'target/site/', reportFiles: 'surefire-report.html', reportName: 'HTML Report', reportTitles: ''])
        }
        
        stage('Build Docker Image'){
            echo "Building docker image for addressbook application ..."
            sh "${dockerCMD} build -t aniketute/springboot:${tagName} ."
        }
        
        stage("Push Docker Image to Docker Registry"){
            echo "Pushing image to docker hub"
            //withCredentials([string(credentialsId: 'dockerPwd', variable: 'dockerHubPwd')])
            withCredentials([usernamePassword(credentialsId: 'dockerPwd', passwordVariable: 'dockerHubPwd', usernameVariable: 'dockerHubUsername')]) {
            sh "${dockerCMD} login -u aniketute -p ${dockerHubPwd}"
            sh "${dockerCMD} push aniketute/springboot:${tagName}"
            }
        }
        
        stage('Deploy Application'){
            echo "Installing desired software.."
            echo "Bring docker service up and running"
            echo "Deploying addressbook application"
            ansiblePlaybook credentialsId: 'root', disableHostKeyChecking: true, installation: 'ansible 2.9.22', inventory: '/etc/ansible/hosts', playbook: 'deploy-playbook.yml'
        }
        
        stage('Clean up'){
            echo "Cleaning up the workspace..."
            cleanWs()
        }
    }
}
catch(Exception err){
    echo "Exception occured..."
    currentBuild.result="FAILURE"
    emailext body: 'Hi Team, \n\n Please go to ${BUILD_URL} for more details and verify the cause for the build failure.', subject: 'Build Result - ${JOB_NAME} ${BUILD_NUMBER}', to: 'aniket.utekar017@gmail.com'
}
finally {
    (currentBuild.result!= "ABORTED") || (currentBuild.result!= "FAILURE")  && node("master") {
        echo "finally gets executed and end an email notification for every build"
        emailext body: 'Hi Team, \n\n Your build has been successful. \n\n Please go to ${BUILD_URL} for more details', subject: 'Build Result - ${JOB_NAME} ${BUILD_NUMBER}', to: 'aniket.utekar017@gmail.com'
    }
    
}
