pipeline{
    agent any
    parameters {
        extendedChoice bindings: '', description: 'dockimage版本号', 
        groovyClasspath: '', 
        groovyScript: '''
            def service='bit'
            def docker_tag=["/bin/bash","-c","""aws ecr list-images --repository-name $service | jq .imageIds[].imageTag | tr -d '"'|sort -nr -t'_' -k2  |xargs echo |sed -e 's/ /,/g' """]
            return docker_tag.execute().text
            ''', 
            multiSelectDelimiter: ',', name: 'docker_tag', quoteValue: false, saveJSONParameterToFile: false, type: 'PT_MULTI_SELECT', visibleItemCount: 1
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
                    // 服务env
                    env.jar_name = "$JOB_NAME".split('-',3)[2]
                    def SERVICE_CONFIG_ALL = readJSON file: "/opt/config/service.json"
                    def SERVICE_CONFIG = SERVICE_CONFIG_ALL["$jar_name"]
                    env.jar_port = SERVICE_CONFIG["jar_port"]
                    println "$SERVICE_CONFIG"
                    println "jar_name:$jar_name"
                    println "jar_port:$jar_port"
                }
            }
        }
        stage('deploy'){
            steps{
                script{
                    switch(jar_name){
                        case "app-ws":
                            type="ws"
                            break;
                        case "app-web":
                            type="web"
                            break;
                        case "server-trading-clear":
                            type="clear"
                            break;  
                        case "server-trading":
                            type="trading"
                            break;  
                        default:
                            type="deploy"
                            break;
                    }
                }
                ansiblePlaybook become: true, 
                credentialsId: 'a7d5b2f0-03c8-4fa7-bf70-b5a148134c1a', 
                disableHostKeyChecking: true, 
                extras: '--extra-vars hosts=stg-$jar_name --extra-vars jar_name=$jar_name --extra-vars jar_port=$jar_port --extra-vars docker_tag=$docker_tag', 
                inventory: '/etc/ansible/hosts', 
                playbook: "/etc/ansible/yml/${type}.yml"  
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