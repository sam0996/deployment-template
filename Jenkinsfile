podTemplate(yaml:"""
spec:
  containers:
  - name: jnlp
    volumeMounts:
    - name: home-dir
      mountPath: /home/jenkins
    env:
    - name: ARTIFACTORY_URL
      valueFrom:
        configMapKeyRef:
          name: artifactory-config
          key: url
    - name: ARTIFACTORY_USERNAME
      valueFrom:
        secretKeyRef:
          name: artifactory-credentials
          key: username
    - name: ARTIFACTORY_PASSWORD
      valueFrom:
        secretKeyRef:
          name: artifactory-credentials
          key: password
  - name: maven
    tty: true
    command: ["/bin/bash"]
    volumeMounts:
    - name: home-dir
      mountPath: /home/jenkins
    image: maven:3.6.3-jdk-8
  volumes:
  - name: home-dir
    emptyDir: {}
  
""") {
    node(POD_LABEL) {
        // create Artifactory server instance
        def server = Artifactory.newServer url: env.ARTIFACTORY_URL, username: env.ARTIFACTORY_USERNAME, password: env.ARTIFACTORY_PASSWORD
        // Create an Artifactory Maven instance.
        def rtMaven = Artifactory.newMavenBuild()
        def buildInfo

        stage('Clone') {
          checkout scm
        }

        container("maven") {
          stage('Set up maven') {
            sh"""
              cp -r /usr/share/maven /home/jenkins
            """
          }
        }

        stage('Artifactory configuration') {
          // Tool name from Jenkins configuration
          // rtMaven.tool = "Maven-3.3.9"
          env.MAVEN_HOME = '/home/jenkins/maven'
          print "${env.PATH}"
          // Set Artifactory repositories for dependencies resolution and artifacts deployment.
          rtMaven.deployer releaseRepo:'example-repo-local', server: server
          // rtMaven.resolver releaseRepo:'libs-release', snapshotRepo:'libs-snapshot', server: server
          sh"""
          echo "PATH=$MAVEN_HOME/bin:$PATH" >> ./env-config
          """
        }

        stage('Maven build') {
          dir("maven-example") {
            buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install'
          }
        }

        stage('Publish build info') {
          server.publishBuildInfo buildInfo
        }
    }
}