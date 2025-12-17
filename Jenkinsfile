def registry = 'https://taxi41.jfrog.io/artifactory'
pipeline {
    agent {
        node {
            label 'maven'
        }
    }
environment {
    PATH = "/opt/apache-maven-3.9.11/bin:$PATH"
    (SONAR_TOKEN = credentials('SONAR_TOKEN'))
    
}
   stages {
        stage("build"){
            steps {
                 echo "----------- build started ----------"
                sh 'mvn package'
                 echo "----------- build complted ----------"
            }
        }
        stage("test"){
            steps{
                echo "----------- unit test started1 ----------"
                sh 'mvn surefire-report:report'
                 echo "----------- unit test Complted ----------"
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    // Run SonarQube analysis
                    sh """
                    mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:3.9.1.2184:sonar \
                    -Dsonar.projectKey=taxi-app41_taxi \
                    -Dsonar.organization=taxi-app41 \
                    -Dsonar.host.url=https://sonarcloud.io \
                    -Dsonar.token=${SONAR_TOKEN}
                    """
                }
            }
        }
    stage("Jar Publish") {
        steps {
            script {
                    echo '<--------------- Jar Publish Started --------------->'
                     def server = Artifactory.newServer url:registry,  credentialsId:"jfrog-cred"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "/home/ubuntu/jenkins/workspace/taxi-booking/taxi-booking/target/(*)",
                              "target": "taxi41-libs-release-local/{1}",
                              "flat": "false",
                              "props" : "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
                            }
                         ]
                     }"""
                     def buildInfo = server.upload(uploadSpec)
                     buildInfo.env.collect()
                     server.publishBuildInfo(buildInfo)
                     echo '<--------------- Jar Publish Ended --------------->'  
             }
        }   
    }


}
}
