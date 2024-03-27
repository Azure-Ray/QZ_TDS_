import os

# 获取当前用户的主目录
home_directory = os.path.expanduser("~")

# 在主目录下创建一个子目录用于下载文件
download_folder = os.path.join(home_directory, "downloads_from_selenium")

# 如果下载目录不存在，则创建它
if not os.path.exists(download_folder):
    os.makedirs(download_folder)

print(f"文件将被下载到：{download_folder}")
