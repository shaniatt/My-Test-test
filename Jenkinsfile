def version = params['mavenVersion'] ?: '3.6.2'
def opts = params['opts'] ?: ['-B', '-T', '1C', '-U', '-s', 'settings.xml']
def goals = params['goals'] ?: ['clean', 'install']
def pom = params['pom'] ?: 'pom.xml'
def maven_opts = [
    '-Xmx2g',
    '-Dmaven.artifacts.threads=5'
  ]


node {
    def server = Artifactory.server 'artifactory'
    def buildInfo = Artifactory.newBuildInfo()
    buildInfo.env.collect()

    def rtMaven = Artifactory.newMavenBuild()
    rtMaven.tool = "MAVEN_TOOL"
    rtMaven.resolver server: server,
                     releaseRepo: 'libs-release',
                     snapshotRepo: 'libs-snapshot'

    rtMaven.deployer server: server,
                     releaseRepo: 'libs-release-local',
                     snapshotRepo: 'libs-snapshot-local'

    rtMaven.deployer.deployArtifacts = true
    
    deleteDir()

    stage ('checkout') {
      checkout scm

      /** Set the job display to the current pom version **/
      currentBuild.displayName = readMavenPom(file: pom).version
    }    


    def settings = configFile(fileId: 'cac29fc6-26cf-4498-87de-4f88242ca1e7',
                              targetLocation: 'settings.xml',
                              replaceTokens: true)    
    
    configFileProvider([settings]) {
      withMaven(maven: "MAVEN_TOOL",
                mavenLocalRepo: "${WORKSPACE}/.m2",
                mavenOpts: maven_opts.join(' ')) {
        goals.each { target ->
          stage (target) {
            rtMaven.run pom: pom,
                        goals: "${target} ${opts.join(' ')}".toString(),
                        buildInfo: buildInfo
          }
        }
      }
    }
    
    server.publishBuildInfo buildInfo
    
}
