pipeline {
    agent any
    
    tools {
        git 'git'
        jdk 'JDK'
        maven 'maven'
        nodejs 'nodejs'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }


    stages {
        
        stage('Clean WorkSpace') {
            steps {
                cleanWs()
            }
        }
        
        stage ('Docker_Image_CleanUp') {
            steps {
                sh 'docker image prune --all --filter "until=1h" -f'
            }
        }

        stage('Git CheckOut') {
            steps {
                git branch: 'main', url: 'https://github.com/Raghava0684/Zomato-Clone-code.git'
            }
        }
        
        stage('SonarQube-Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=zomato \
                    -Dsonar.projectKey=zomato '''
                }
            }
        }
        
        stage('Wait For Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-Cred'
            }
        }
        
        stage ('Install npm Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        
        stage ('OWASP-Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'Dependency-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage ('trivy FS scanning') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        
        stage ("Docker Build && Push") {
            steps {
                script {
                 
                withDockerRegistry(credentialsId: 'Docker-Cred', toolName: 'docker') {
                
                    sh "docker build -t  raghava017/zomato:${BUILD_NUMBER} ."
                    sh "docker push raghava017/zomato:${BUILD_NUMBER} "
                    }
                    
                }
            }
        }
        
        
        stage ("Scan Docker Image With Trivy") {
            steps {
                sh 'trivy image raghava017/zomato:latest > trivy.txt'
            }
        }
        
        stage ("Deploy Docker Image ") {
            steps {
                sh "docker run -d --name zomato -p 3000:3000 raghava017/zomato:latest"
            }
        }
    }
    
    post {
        always {
            echo 'This will always run'
        }
        success {
            mail to: 'ajayreddimalla123@gmail.com',
                 subject: "Jenkins Pipeline Status: SUCCESSFUL",
                 body: "Your Jenkins Build pipeline has been completed successfully."
        }
        failure {
            mail to: 'ajayreddimalla123@gmail.com',
                 subject: "Jenkins Pipeline Status: FAILED",
                 body: "Your Jenkins Build pipeline hasbeen failed. Please check the logs for more information."
        }
    }
    
    
}
