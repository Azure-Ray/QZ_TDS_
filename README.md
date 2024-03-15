

pipeline {
    agent any
    stages {
        stage('Check Other Job Status in Last 24 Hours') {
            steps {
                script {
                    def otherJobName = 'YourOtherJobName' // 替换为你需要检查的Job名称
                    def otherJob = Jenkins.instance.getItemByFullName(otherJobName)
                    def twentyFourHoursAgo = new Date() - 1
                    def foundSuccessfulBuild = otherJob.builds.find {
                        it.getResult() == hudson.model.Result.SUCCESS &&
                        it.getTimestamp().after(twentyFourHoursAgo)
                    }
                    if (!foundSuccessfulBuild) {
                        echo "在过去24小时内没有找到成功的构建，终止执行。"
                        currentBuild.result = 'ABORTED'
                        return
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    // 如果前一步骤未终止流水线，则继续部署
                    def commonStage = new commonDeployStage()
                    commonStage.runCommonDeployStage(this, DEV_PROFILE, COPY_DEPLOY_RESOURCE)
                }
            }
        }
    }
}
