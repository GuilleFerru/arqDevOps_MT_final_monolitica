pipeline {
    agent {
        docker {
            image 'maven:3.8.8-eclipse-temurin-17-alpine'
        }
    }
    stages {
        
        // stage('Checkout SCM') {
        //     steps {
        //         git branch: 'master', url: 'https://github.com/GuilleFerru/arqDevOps_MT_final_monolitica.git'
        //     }
        // }

        stage('Validate') {
            steps {
                sh 'mvn validate -B -ntp'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile -B -ntp'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn clean test -B -ntp'
            }
            post {
                success {
                    junit 'target/surefire-reports/*.xml'
                    jacoco(execPattern: 'target/jacoco.exec')
                }
            }
        }

        stage('Verify') {
            steps {
                sh 'mvn verify -B -ntp'
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests -B -ntp'
            }
        }

        stage('SonarQube') {
            steps {
                script {
                    def branch = env.GIT_BRANCH?.split('/')?.last() ?: 'master'
                    echo "branch: ${branch}"
                    withSonarQubeEnv('sonarqube') {
                        // Ejecuta el análisis de SonarQube
                        sh "mvn sonar:sonar -Dsonar.branch.name=${branch} -B -ntp"
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    // Esperar el resultado del Quality Gate de SonarQube
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Artifactory') {
            steps {
                script {
                    // Forma 1 - Utilizando rtMaven
                    sh 'env | sort'
                    env.MAVEN_HOME = '/usr/share/maven'

                    def releases = 'spring-petclinic-rest-release'
                    def snapshots = 'spring-petclinic-rest-snapshot'

                    def server = Artifactory.server 'artifactory'
                    def rtMaven = Artifactory.newMavenBuild()
                    rtMaven.deployer server: server, releaseRepo: releases, snapshotRepo: snapshots
                    def buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install -B -ntp -DskipTests'

                    server.publishBuildInfo buildInfo
                }
            }
        }
    }

    post {
        success {
            // Archivar los artefactos generados y limpiar el workspace
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            cleanWs()
        }
    }
}
