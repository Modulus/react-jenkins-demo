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
       def versionNumber = gitCommit.substring(0,10)
       stage('Run tests') {
            container("node"){
                stage("Prepare and run tests"){
                    sh """
                    npm cache verify  
                    npm install     
                    ## Installing junit-report support
                    npm install --save-dev jest-junit


                    npm test
                    """
                }
            }
        }
         stage("Build container"){
            container("docker"){
                sh """
                ls -la
                """
                docker.withRegistry("https://tv2norge-docker-test-local.jfrog.io", "artifactory"){
                    def builtImage = docker.build("tv2norge-docker-test-local.jfrog.io/tv2sumo-advertising-administration:1.0.${versionNumber}-${gitBranch}")   
                    //builtImage.push()

                    slackSend  baseUrl: "https://tv2sumo.slack.com/services/hooks/jenkins-ci/", channel: "#i2-deploy", color: "good", message: "tv2norge-docker-test-local.jfrog.io/tv2sumo-advertising-administration:1.0.${versionNumber}-${gitBranch} has been uploaded to artifactory", tokenCredentialId: "slack"

                }
              
            }
        }
    }
}