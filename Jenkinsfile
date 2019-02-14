// this guarantees the node will use this template
def label = "mypod-${UUID.randomUUID().toString()}"
podTemplate(podRetention: never(), label: label, containers : [
    containerTemplate( name: "node", image: "node:8", ttyEnabled: true, command: "cat"),
    containerTemplate( name: "docker", image: "docker:18.09-dind", command: "cat", ttyEnabled: true, privleged: false)
    ],
    volumes: [hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')]) {
    node(label) {
       def repo = checkout scm
       def gitCommit = repo.GIT_COMMIT
       def gitBranch = repo.GIT_BRANCH
       def commitHash = gitCommit.substring(0,10)
       stage('Run tests') {
            container("node"){
                stage("Prepare and run tests"){
                    sh """
                    npm cache verify  
                    npm install     
                    npm test


                    echo 'The tests for this repo does not work with reports..... Not my fault'
                    """
                }
                stage("Publish junit reports"){
                    junit "junit.xml"
                }
            }
        }
         stage("Build container"){
            container("docker"){
                sh """
                ls -la
                """
                docker.withRegistry("https://tv2norge-docker-test.jfrog.io", "artifactory"){
                    def imageName = "tv2norge-docker-test.jfrog.io/interaktiv-react-jenkins-demo:1.0.${env.BUILD_NUMBER}-${gitBranch}-${commitHash}"
                    def builtImage = docker.build("${imageName}")   
                    builtImage.push()

                    slackSend  baseUrl: "https://tv2sumo.slack.com/services/hooks/jenkins-ci/", channel: "#i2-deploy", color: "good", message: "${imageName} has been built, but not pushed", tokenCredentialId: "slack"

                }
              
            }
        }
    }
}