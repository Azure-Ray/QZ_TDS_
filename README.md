def last24Hrs = new Date() - 1
                    def successBuildLast24Hrs = job.builds.findAll { build ->
                        build.result == hudson.model.Result.SUCCESS && build.timestamp.timeInMillis > last24Hrs.time
                    }
                    if (successBuildLast24Hrs.isEmpty()) {
                        error("在过去24小时内未找到成功的构建.")
                    }
