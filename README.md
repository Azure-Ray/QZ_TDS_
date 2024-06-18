#!/bin/bash

# 要处理的分支名称数组
BRANCHES_TO_TAG=("branch_name1" "branch_name2" "branch_name3")
# 标签前缀名称
TAG_PREFIX="tag_prefix"

# 遍历每个分支
for BRANCH_TO_TAG in "${BRANCHES_TO_TAG[@]}"; do
    # 标签名称可以使用分支名称或添加前缀
    TAG_NAME="${TAG_PREFIX}_${BRANCH_TO_TAG}"

    # 检查分支是否存在
    if git show-ref --verify --quiet refs/heads/$BRANCH_TO_TAG; then
        # 创建标签
        git tag $TAG_NAME $BRANCH_TO_TAG

        # 推送标签到远程仓库
        git push origin $TAG_NAME

        # 删除本地分支
        git branch -d $BRANCH_TO_TAG

        # 删除远程分支
        git push origin --delete $BRANCH_TO_TAG

        echo "Branch '$BRANCH_TO_TAG' has been tagged as '$TAG_NAME' and deleted."
    else
        echo "Branch '$BRANCH_TO_TAG' does not exist."
    fi
done
