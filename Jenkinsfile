def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
    'UNSTABLE': 'warning'
]

pipeline {
    agent any

    tools {
        maven "MAVEN3.9"
        jdk "JDK17"
    }
    
    environment {
          MAVEN_OPTS = "--add-opens=jdk.compiler/com.sun.tools.javac.processing=ALL-UNNAMED --add-opens=jdk.compiler/com.sun.tools.javac.util=ALL-UNNAMED --add-opens=jdk.compiler/com.sun.tools.javac.file=ALL-UNNAMED --add-opens=jdk.compiler/com.sun.tools.javac.parser=ALL-UNNAMED --add-opens=jdk.compiler/com.sun.tools.javac.tree=ALL-UNNAMED --add-opens=jdk.compiler/com.sun.tools.javac.api=ALL-UNNAMED --add-opens=jdk.compiler/com.sun.tools.javac.code=ALL-UNNAMED --add-opens=jdk.compiler/com.sun.tools.javac.comp=ALL-UNNAMED --add-opens=jdk.compiler/com.sun.tools.javac.jvm=ALL-UNNAMED --add-opens=jdk.compiler/com.sun.tools.javac.main=ALL-UNNAMED --add-opens=jdk.compiler/com.sun.tools.javac.model=ALL-UNNAMED --add-opens=jdk.compiler/com.sun.tools.javac.tree=ALL-UNNAMED"
    }


    stages {

        stage('Fetch code') {
            steps {
                git branch: 'main', url: 'https://github.com/YassineBerrada/spring2'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving it...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        //stage('UNIT TEST') {
        //    steps {
        //        sh 'mvn test'
        //    }
    //    }

        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Sonar Code Analysis') {
            environment {
                scannerHome = tool 'sonar6.2'
            }
            steps {
                withSonarQubeEnv('sonarserver') {
                    sh '''${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=vprofile \
                        -Dsonar.projectName=vprofile \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports/ \
                        -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                    '''
                }
            }
        }

        //stage('Quality Gate') {
        //    steps {
        //        timeout(time: 1, unit: 'HOURS') {
        //            waitForQualityGate abortPipeline: true
        //        }
        //    }
    //    }

        stage('Docker Build & Push') {
            environment {
                IMAGE_NAME = "vprofile"
                IMAGE_TAG = "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}"
                NEXUS_DOCKER_REPO = "3.88.248.130:8082"
                DOCKER_REGISTRY = "${NEXUS_DOCKER_REPO}/${IMAGE_NAME}"
            }
            steps {
                script {
                    sh """
                        echo "YassineNexus12**" | docker login $NEXUS_DOCKER_REPO -u admin --password-stdin
                        docker build -t $DOCKER_REGISTRY:$IMAGE_TAG .
                        docker push $DOCKER_REGISTRY:$IMAGE_TAG
                        docker logout $NEXUS_DOCKER_REPO
                    """
                }
            }
        }
        stage('Deploy with Ansible') {Add commentMore actions
    steps {
        ansiblePlaybook(
            playbook: 'ansible/deploy_docker.yml',
            inventory: 'ansible/hosts.ini',
            credentialsId: 'sonarqube3'
        )
}}



    }

    post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#devopscicd',
                      color: COLOR_MAP[currentBuild.currentResult],
                      message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}"
        }
    }
}