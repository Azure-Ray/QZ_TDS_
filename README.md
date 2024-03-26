from selenium import webdriver
from selenium.webdriver.common.keys import Keys

# 初始化WebDriver
driver = webdriver.Chrome()

# 打开登录页面
driver.get("https://login.microsoftonline.com")

# 找到用户名输入框并输入用户名
username_input = driver.find_element_by_id("i0116")
username_input.send_keys("your_email@example.com")

# 找到下一步按钮并点击
next_button = driver.find_element_by_id("idSIButton9")
next_button.click()

# 等待页面加载，找到密码输入框并输入密码（根据实际情况调整等待方式）
password_input = driver.find_element_by_id("i0118")
password_input.send_keys("your_password")

# 找到登录按钮并点击
sign_in_button = driver.find_element_by_id("idSIButton9")
sign_in_button.click()

# 根据需要处理后续的重定向或二因素认证等步骤
