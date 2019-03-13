// this guarantees the node will use this template
def label = "mypod-${UUID.randomUUID().toString()}"
podTemplate(podRetention: never(), label: label, containers : [
    containerTemplate( name: "node", image: "node:8", ttyEnabled: true, command: "cat"),
    containerTemplate( name: "docker", image: "docker:18.09-dind", command: "cat", ttyEnabled: true, privleged: false)
    ],
    volumes: [hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'), emptyDirVolume( mountPath: "/tmp/build", memory: true)]) {
    node(label) {
       def repo = checkout scm
       def gitCommit = repo.GIT_COMMIT
       def gitBranch = repo.GIT_BRANCH
       def commitHash = gitCommit.substring(0,10)
       def project = "interaktiv-react-jenkins-demo"
      def imageName = "tv2norge-docker-test.jfrog.io/interaktiv-react-jenkins-demo:1.0.${env.BUILD_NUMBER}-${gitBranch}-${commitHash}"

       stage('Run tests') {
            container("node"){
                stage("Prepare and run tests"){
                    sh """
                    npm cache verify  
                    npm install     
                    npm test
                    """
                }
                stage("Publish junit reports"){
                    junit "/tmp/build/junit.xml"
                }
            }
        }
         stage("Build container"){
            container("docker"){
                sh """
                ls -la
                """
                docker.withRegistry("https://tv2norge-docker-test.jfrog.io", "artifactory"){
                    def builtImage = docker.build("${imageName}")   
                    builtImage.push()

                   // slackSend  baseUrl: "https://tv2sumo.slack.com/services/hooks/jenkins-ci/", channel: "#i2-deploy", color: "good", message: "${imageName} has been built, and pushed to artifactory", tokenCredentialId: "slack"

                }
              
            }
        }
        stage("Sonarqube analyzis"){
            container("sonar"){
                stage("Publish results"){

                    withCredentials([string(credentialsId: "sonar-interaktiv-info-agent", variable: "SONAR_KEY")]){
                        sh """
                        ls -la
                        sonar-scanner \
                        -Dsonar.projectKey=interaktiv-info-agent \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://interaktiv-sonar-sonarqube.interaktiv.svc.cluster.local:9000 \
                        -Dsonar.login=${SONAR_KEY} \
                        -Dsonar.junit.reportPaths=/tmp/build/junit.xml
                        """
                    }
       
                }
            }
        }
    }
}