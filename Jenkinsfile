node(""){
    print(scm.getUserRemoteConfigs()[0].getUrl())
    print(env.BRANCH_NAME)
    print(env.TAG_NAME)
    b_name=env.BRANCH_NAME
    t_name=env.TAG_NAME
    VERSION=t_name
    if (VERSION == null){
	VERSION=b_name
    }
    GIT_VERSION=VERSION
    if (t_name != null){
	GIT_VERSION='refs/tags/' + GIT_VERSION
    }
    label = 'latest'
    if (env.TAG_NAME != null){
	label = env.TAG_NAME
    }
    print(label)

    stage("Git checkout"){
	checkout changelog: false, poll: false, scm: [$class: 'GitSCM', branches: [[name: "$GIT_VERSION"]], extensions: [], userRemoteConfigs: [[url: scm.getUserRemoteConfigs()[0].getUrl()]]]
    }
    stage("Build"){
	registry_id = sh(returnStdout: true, script: 'cat /home/jenkins/registry_id.txt').trim()
	print(registry_id)
	sh """
	    docker build -t cr.yandex/${registry_id}/my-diploma-application:${label} .
	    docker push cr.yandex/${registry_id}/my-diploma-application:${label}
	"""
    }
    if (env.TAG_NAME != null){
	stage("deploy"){
	    workspace = sh(returnStdout: true, script: 'cat /home/jenkins/workspace.txt').trim()
	    sh """
	        cd /home/jenkins/qbec-app/ 
		QBEC_YES=true qbec apply ${workspace} --vm:ext-str app_image_name=cr.yandex/${registry_id}/my-diploma-application --vm:ext-str app_image_tag=${env.TAG_NAME}
	    """
	}
    }
}
