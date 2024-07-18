pipeline {
    agent any

    triggers {
        // 定义轮询触发器，这里设置为每5分钟检查一次代码更新
        pollSCM('*/5 * * * *')
    }

    environment {
	    // GIT地址
        GIT_URL = 'git@e.coding.net:lingfliu/ucs/ucs_alg_node_wrapdemo.git'
	    // 版本信息，用当前时间
        VERSION = VersionNumber versionPrefix: 'prod.', versionNumberString: '${BUILD_DATE_FORMATTED, "yyyyMMdd"}.${BUILDS_TODAY}'
        GIT_URL_WITH_AUTH = 'git@e.coding.net:lingfliu/ucs/ucs_alg_node_wrapdemo.git'
        DOCKER_IMAGE = "ucs_alg_node_wrapdemo:${env.BUILD_ID}"
        CODING_DOCKER_REPO = ' lingfliu-docker.pkg.coding.net/ucs/docker'
        DOCKER_IMAGE_TAGGED = " lingfliu-docker.pkg.coding.net/ucs/docker/ucs_alg_node_wrapdemo:${VERSION}"
        CODING_DOCKER_USERNAME = '18219252116'
        CODING_DOCKER_PASSWORD = 'w13302687555'
    }
    
    parameters {
        choice(
		name: 'OP',
		choices: 'publish\nrollback', 
		description: 'publish(发布新版本时选择，部署后自动生成新tag) rollback(回滚时选择，需要同时选择回滚的tag)'
	    )

        choice(
		name: 'DEPLOYENV', 
		choices: 'prod\ntest', description: 'prod(部署环境) test(测试环境)'
	    )

        gitParameter(
		branch: '', 
		branchFilter: 'origin/(.*)', 
		defaultValue: 'master', 
		description: '选择将要构建的标签', 
		name: 'TAG', 
		quickFilterEnabled: false, 
		selectedValue: 'TOP',
		sortMode: 'DESCENDING_SMART', 
		tagFilter: '*',
		type: 'PT_TAG', 
		useRepository: env.GIT_URL
	    )

        string(
            name: 'REMOTE_DIR', 
            defaultValue: '/home/springboot_2',
            description: '远程部署目录'
        )

        string(
            name: 'EXEC_COMMAND', 
            defaultValue: 'nohup sh /home/springboot_2/test.sh', 
            description: '远程部署执行脚本'
        )
    }

    stages {
        stage('拉取代码') {
            steps {
		        // 清除工作空间并拉取最新的代码
                cleanWs()
                // 使用 git 插件拉取代码
                script {
                    if (params.OP == 'publish') {
                        // 如果是发布操作，则拉取 master 分支的最新代码
                        checkout([$class: 'GitSCM', branches: [[name: '*/master']], userRemoteConfigs: [[url: "${env.GIT_URL}"]]])
                    } else if (params.OP == 'rollback') {
                        // 如果是回滚操作，则拉取指定的 tag
                        checkout([$class: 'GitSCM', branches: [[name: "${params.TAG}"]], userRemoteConfigs: [[url: "${env.GIT_URL}"]]])
                    }
                }
            }
        }
        
        stage('Docker Build') {
            steps {
                script {
                    // 构建 Docker 镜像，构建和运行
                   sh ' docker build -t ${DOCKER_IMAGE} .'
                }
            }
        }
        stage('Docker Login') {
            steps {
                script {
                    // 登录到 Coding.net 的 Docker 仓库
                    sh 'echo ${CODING_DOCKER_PASSWORD} | docker login docker.coding.net -u ${CODING_DOCKER_USERNAME} --password-stdin'
                }
            }
        }
        
        stage('Docker Tag') {
            steps {
                script {
                    // 标记 Docker 镜像
                    sh 'docker tag ${DOCKER_IMAGE} ${DOCKER_IMAGE_TAGGED}'
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    // 推送 Docker 镜像到 Coding.net
                    sh 'docker push ${DOCKER_IMAGE_TAGGED}'
                }
            }
        }
        
        
        stage('远程部署项目') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: '123.60.173.147', 
                            transfers: [
                                sshTransfer(
                                    cleanRemote: false,
                                    excludes: '', 
                                    execCommand: params.EXEC_COMMAND, 
                                    execTimeout: 120000, 
                                    flatten: false, 
                                    makeEmptyDirs: false, 
                                    noDefaultExcludes: false, 
                                    patternSeparator: '[, ]+', 
                                    remoteDirectory: params.REMOTE_DIR, 
                                    remoteDirectorySDF: false, 
                                    removePrefix: '', sourceFiles: '*'
                                )
                            ],
                            usePromotionTimestamp: false, 
                            useWorkspaceInPromotion: false, 
                            verbose: false
                        )
                    ]
                )
                 sh ' docker-compose up -d'
            }
        }

        stage('新版本打tag') {
            steps {
                // 执行脚本步骤来打标签并推送
                script {
                    // 打标签
                    sh 'git tag ' + env.VERSION
                    // 推送标签至远程仓库
                    sh 'git push ' + env.GIT_URL_WITH_AUTH + ' ' + env.VERSION
                }
            }
        }
    }
}
