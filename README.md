from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from webdriver_manager.chrome import ChromeDriverManager

# 指定下载文件的默认目录
download_folder = "/path/to/your/download/directory"

chrome_options = Options()
chrome_options.add_experimental_option("prefs", {
  "download.default_directory": download_folder,  # 设置下载路径
  "download.prompt_for_download": False,  # 禁止弹出下载前的确认框
  "download.directory_upgrade": True,  # 设置下载路径时自动创建目录
  "safebrowsing.enabled": True  # 保持安全浏览功能启用
})

# 初始化WebDriver
driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()), options=chrome_options)

# 接下来是你的Selenium操作，如登录、导航、点击下载CSV文件等
