pipeline{
    agent any
    parameters {
        extendedChoice bindings: '', description: 'dockimage版本号', 
        groovyClasspath: '', 
        groovyScript: '''
            def service='bit'
            def docker_tag=["/bin/bash","-c","""aws ecr list-images --repository-name $service | jq .imageIds[].imageTag | tr -d '"' |grep master|sort -nr -t'-' -k1|xargs echo |sed -e 's/ /,/g' """]
            return docker_tag.execute().text
            ''', 
            multiSelectDelimiter: ',', name: 'docker_tag', quoteValue: false, saveJSONParameterToFile: false, type: 'PT_SINGLE_SELECT', visibleItemCount: 1
    }
    environment {
        def registry = '425194982185.dkr.ecr.ap-southeast-1.amazonaws.com'
    }
    stages{
        stage('json env'){
            steps{
                wrap([$class: 'BuildUser']) {
                    script {
                        env.user = "$BUILD_USER "
                        println(user)
                    }
                }
                script{
                    env.item = "$JOB_NAME".split('-',3)[0] + '-' + "$JOB_NAME".split('-',3)[1]
                    env.ENV = "$JOB_NAME".split('-',3)[0]
                    env.project = "$JOB_NAME".split('-',3)[2]
                    env.deploy = "$ENV" + '-' + "$project"
                    env.ecr = "bit-$PROJECT"     
                }
                configFileProvider([configFile(fileId: 'pipeline-ns', variable: 'nsconfig')]) {
                    script{
                        def NS_CONFIG = readJSON file: "${nsconfig}"
                        env.ns = NS_CONFIG["$item"]['ns']
                    }
                }
            }
        }
        stage('deploy'){
            steps{
                sh'''
                    kubectl -n $ns set image deployment/$deploy  $deploy=$registry/$ecr:$docker_tag --record
                '''
            }
        }
        stage('shorttext'){
            steps{
                script {
                    env.shorttext = "发布人:$user 发布版本:$docker_tag"
                    manager.addShortText(shorttext)
                }
            }
        }
    }
}