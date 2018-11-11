  try {
     timeout(time: 20, unit: 'MINUTES') {
        def appName="demo-secondary"
        def artifactName="${appName}-mule-application"
        def project=""
        node {
          stage("Initialize") {
            project = env.PROJECT_NAME
          }
        }
        node("maven") {
          stage("Checkout"){
            git url: "https://github.com/jam01/mul4-demo-secondary", branch: "master"
          }
          stage("Build JAR") {
            sh "mvn clean package -DskipTests -Dmaven.javadoc.skip=true -Dmaven.site.skip=true -Dmaven.source.skip=true -Djacoco.skip=true -Dcheckstyle.skip=true -Dfindbugs.skip=true -Dpmd.skip=true -Popenshift -e -B"
            stash name:"app", includes:"**/target/*.jar"
          }
          stage("Unit Tests"){
            sh "mvn clean test"
          }
        }
        node {
          stage("Build Image") {
            unstash name:"app"
            sh "oc start-build ${appName}-runtime-build -n ${project} --from-file=target/${artifactName}.jar --follow"
            timeout(time: 20, unit: 'MINUTES') {
              openshift.withCluster() {
                openshift.withProject() {
                  def bc = openshift.selector('bc', "${appName}-runtime-build")
                  echo "Found ${bc.count()} ${appName}-runtime-build"
                  def blds = bc.related('builds')
                  blds.untilEach {
                    return it.object().status.phase == "Complete"
                  }
                }
              }
            }
          }
          stage("Integration Tests"){
            echo "Testing!"
          }
          stage("Deploy") {
            openshift.withCluster() {
              openshift.withProject() {
                def dc = openshift.selector('dc', "${appName}")
                dc.rollout().status()
              }
            }
          }
        }
     }
  } catch (err) {
     echo "in catch block"
     echo "Caught: ${err}"
     currentBuild.result = 'FAILURE'
     throw err
  }
