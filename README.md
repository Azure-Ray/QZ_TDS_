@Library('PipelineAsCodeMod')
def globalVars.JAVA_HOME = "/opt/build tools/jdk/17.0.2"

pipeline {
    agent any
    environment {
        // 将你的变量放在这里
        DISABLE_CHECKMARX = 'true'
        DISABLE_IQ = 'true'
        DISABLE_SONAR = 'true'
        ENABLE_CYBERFLOWS = 'false'
        CYBERFLOWS_CONFIG_ID = '2bkJpIdMhRQ5kudp90PxrDqEs'
    }
    stages {
        stage('Check Other Job Status in Last 24 Hours') {
            steps {
                script {
                    // 替换"OtherJobName"为你想检查的Job的名称
                    def otherJob = Jenkins.instance.getItemByFullName("OtherJobName")
                    def builds = otherJob.getBuilds()
                    def foundSuccessfulBuild = false
                    def twentyFourHoursAgo = new Date() - 24

                    for (def build : builds) {
                        def buildTimestamp = build.getTimestamp()
                        def buildDate = new Date(buildTimestamp.timeInMillis)
                        if (buildDate.after(twentyFourHoursAgo) && build.result == hudson.model.Result.SUCCESS) {
                            foundSuccessfulBuild = true
                            break
                        }
                    }

                    if (foundSuccessfulBuild) {
                        echo '在过去24小时内找到成功的构建，继续执行。'
                    } else {
                        echo '在过去24小时内没有找到成功的构建，终止执行。'
                        // 这里我们直接结束当前构建，将其标记为成功
                        currentBuild.result = 'SUCCESS'
                        // 使用下面的方法可以将当前构建标记为失败
                        // error('在过去24小时内没有找到成功的构建。')
                    }
                }
            }
        }
        stage('Normal Build') {
            steps {
                script {
                    // 在这里调用你的构建命令
                    def commonStage = new commonBuildStage()
                    commonStage.runCommonBuildStage(this, VAR_MVN_RELEASE_UPDATE_VERSION_STEPS)
                }
            }
        }
    }
}
