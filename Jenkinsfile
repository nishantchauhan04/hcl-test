parameters {
        string(defaultValue: 'https://registry.hub.docker.com', description: '', name: 'RegistryURL')
	string(defaultValue: '/', description: '', name: 'tagsep')
	string(defaultValue: 'Test', description: '', name: 'AppName')
	string(defaultValue: '80', description: '', name: 'AppPort')
	string(defaultValue: 'micro-system', description: '', name: 'NameSpace')
	string(defaultValue: 'helm-cred-repo-id', description: '', name: 'HelmCredId')
}
podTemplate(
    label: 'mypod', 
    inheritFrom: 'default',
    containers: [
        containerTemplate(
            name: 'docker', 
            image: 'docker:18.02',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'helm', 
            image: 'lachlanevenson/k8s-helm:v2.11.0',
            ttyEnabled: true,
            command: 'cat'
        )
    ],
    volumes: [
        hostPathVolume(
            hostPath: '/var/run/docker.sock',
            mountPath: '/var/run/docker.sock'
        )
    ]
) 
{
    node('mypod') {
        def commitId
        stage ('Extract') {
            checkout scm
            commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
			checkout([$class: 'GitSCM', 
				branches: [[name: '*/master']], 
				doGenerateSubmoduleConfigurations: false, 
				extensions: [[$class: 'CleanCheckout'],[$class: 'RelativeTargetDirectory', relativeTargetDir: 'install']], 
				submoduleCfg: [], 
				userRemoteConfigs: [[credentialsId: "${params.HelmCredId}", url: 'https://github.com/nishantchauhan04/java.git']]
			])
			sh 'ls -ltr'
        }
		stage ('Docker') {
            container ('docker') {
			    withDockerRegistry([credentialsId: 'dockerhub']) {
					sh "docker pull nishantchauhan/javaalpine"
					sh "docker tag nishantchauhan/javaalpine nishantchauhan/javaalpine:${env.BUILD_NUMBER}"
					sh "docker push nishantchauhan/javaalpine:${env.BUILD_NUMBER} "
				}
			    
            }
        }
        stage ('Deploy') {
            container ('helm') {
                sh "helm init --client-only --skip-refresh"
		sh "helm ls"    
                sh "helm upgrade --install --namespace micro-system --wait --set service.identifier=abcd,service.port=80,service.name=Test,image.repository=nishantchauhan/javaalpine,image.tag=${env.BUILD_NUMBER} Test install/base/install/helm -f Values.yaml"
			}
        }
    }
	
}
