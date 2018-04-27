def label = "worker-${UUID.randomUUID().toString()}"

def sha1 = 'master'

podTemplate(cloud: 'tess-k8s', label: label, containers: [
  containerTemplate(
    name: 'jnlp',
    image: 'ecr.vip.ebayc3.com/shutl/payment-service-jenkins',
    args: '${computer.jnlpmac} ${computer.name}',
    ttyEnabled: true,
    envVars: [
      containerEnvVar(key: 'JAVA_OPTS', value: '-Xms1024m -Xmx4196m')
    ],
    alwaysPullImage: true,
    workingDir: '/home/jenkins'
  )
],
volumes: [
  emptyDirVolume(mountPath: '/home/jenkins/workspace', memory: false),
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
],
annotations: [
    podAnnotation(key: "scheduler.alpha.kubernetes.io/tolerations", value: '[{"key":"dedicated","value":"ciaas"}]')
]) {

  node(label) {
    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'payment_service_aws_credentials',
    usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {

      withEnv(["LEIN_USERNAME=$USERNAME",
               "LEIN_PASSWORD=$PASSWORD",
               "GIT_BRANCH=${sha1}"]) {

        def COMPOSE_CONFIG = 'docker-compose-ci.yml'

        stage('setup'){
          echo "Building branch ${sha1}"
          checkout([$class: 'GitSCM', branches: [[name: "${sha1}"]], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'LocalBranch', localBranch: '']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'test_deploy_key', url: 'git@github.com:shutl/payment_service.git']]])
          sh "git status"
        }
      }
    }
  }
}
