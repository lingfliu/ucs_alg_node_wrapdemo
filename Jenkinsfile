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
        //打包镜像相关信息
        DOCKER_IMAGE = "ucs_alg_node_wrapdemo:${VERSION}"
    }
    
    parameters {
        choice(
		name: 'OP',
		choices: 'publish\nrollback', 
		description: 'publish(发布新版本时选择，部署后自动生成新tag) rollback(回滚时选择，需要同时选择回滚的tag)'
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
        
        stage('停止和删除相关容器') {
            steps {
                script {
                    //停止运行中的容器
                    sh 'docker rm -f ucs_alg_node_web | true'
                    //删除相关无用镜像，避免占有服务器资源
                    sh 'docker rmi -f $(docker images -q --filter "reference=ucs_alg_node_wrapdemo*") | true'
                }
            }
        }

        stage('Docker构建镜像') {
            steps {
                script {
                    // 构建 Docker 镜像，构建和运行
                   sh ' docker build -t ${DOCKER_IMAGE} .'
                }
            }
        }
    

        stage('配置环境变量') {
            steps {
                script {
                    sh 'echo "VERSION=' + env.VERSION + '" > .env'
                }
            }
        }

        stage('停止相关容器') {
            steps {
                script {
                    //停止运行中的容器
                    sh 'docker rm -f ucs_alg_node_wrapdemo | true'
                }
            }
        }

        stage('部署项目') {
            steps {
                script {
                    sh 'chmod +x ./docker-compose.yaml && docker-compose -f ./docker-compose.yaml up -d'
                }
            }
        }

        stage('删除未被使用的相关镜像和none的镜像') {
            steps {
                script {
                    //删除相关无用镜像，避免占有服务器资源
                    sh 'docker image prune --filter "label=repository=ucs_alg_node_wrapdemo*" | true'
                    //为none的异常镜像
                    sh 'docker rmi $(docker images -f "dangling=true" -q) | true'
                }
            }
        }
    }
}
