pipeline {
    agent any
    parameters {
        string 'DATE'
        string 'TIME'
        choice choices: ['open', 'close'], name: 'STATUS'
    }
    stages {
        stage('1'){
            when {
                not{
                  allOf {
                    environment name: 'DATE', value: ''
                    environment name: 'TIME', value: ''  
                    }
                }
            }

            steps{
                println 'hello'
            }
        }
        stage('2'){
            when {
                not {
                    environment name: 'STATUS', value: ''
                }
            }
            steps{
                sh'''
                RESUMETIME="`date -d "$DATE $TIME"  "+%s"`000"
                aws elbv2 modify-rule --rule-arn arn:aws:elasticloadbalancing:ap-southeast-1:724162388587:listener-rule/app/maintain/bfa91b35dc84ba5f/722386599f1022fb/48d579c59dbbf8f2 \
                        --conditions Field=host-header,Values='www.example.com,api.example.com,example.com' Field=path-pattern,Values='/status' \
                        --actions \
                            \'[{
                            "Type": "fixed-response",
                            "FixedResponseConfig": {
                            "MessageBody": "{ \\"code\\": 0, \\"state\\": \\"'$STATUS'\\",\\"resumeTime\\": \\"'$RESUMETIME'\\" }",
                            "StatusCode": "200",
                            "ContentType": "application/json"
                            }
                            }]\'
                '''
            }
        }
    }
}