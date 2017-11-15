podTemplate(label: 'mypod', containers: [
    containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.8.0', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:latest', command: 'cat', ttyEnabled: true)
  ],
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
  ]) {
    node('mypod') {
        stage('Helm Init') {
            container('helm') {
               sh "helm init --client-only" 
            }
        }
        
        stage('Install Nexus') {
            container('helm') {
               sh "helm delete --purge nexus" 
               sh "helm install --name nexus stable/sonatype-nexus --namespace cd-pipeline"
            }
            
            container('kubectl') {
               waitForAllPodsRunning('cd-pipeline') 
            }
        }
    }
}

def waitForAllPodsRunning(String namespace) {
    timeout(60000) {
        while (true) {
            podsStatus = sh(returnStdout: true, script: "kubectl --namespace='${namespace}' get pods --no-headers").trim()
            def notRunning = podsStatus.readLines().findAll { line -> !line.contains('Running') }
            if (notRunning.isEmpty()) {
                echo 'All pods are running'
                break
            }
            sh "kubectl --namespace='${namespace}' get pods"
            sleep 10
        }
    }
}
