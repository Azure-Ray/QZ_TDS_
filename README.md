from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import TimeoutException

# 配置Chrome的无痕模式
chrome_options = Options()
chrome_options.add_argument("--incognito")

# 指定chromedriver的路径
driver = webdriver.Chrome(executable_path='C:/users/sss/ccc/aaa/chromedriver.exe', options=chrome_options)

# 打开Microsoft登录页面
driver.get("https://login.microsoftonline.com")

# 输入邮箱并提交
email_input = WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.ID, "i0116")))
email_input.send_keys("your_email@example.com" + Keys.ENTER)

# 输入密码
password_input = WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.ID, "i0118")))
password_input.send_keys("your_password" + Keys.ENTER)

# 点击登录按钮
login_button = WebDriverWait(driver, 10).until(EC.element_to_be_clickable((By.ID, "idSIButton9")))
login_button.click()

# 处理“保持登录状态”提示
try:
    stay_signed_in_button = WebDriverWait(driver, 10).until(EC.element_to_be_clickable((By.ID, "idSIButton9")))
    stay_signed_in_button.click()
except TimeoutException:
    print("No 'Stay signed in?' prompt found.")

# 点击i标签的按钮来打开下拉框选项
i_button = WebDriverWait(driver, 10).until(EC.element_to_be_clickable((By.CSS_SELECTOR, "i.icon-menu.col-menu.list.header.context.list-column-icon")))
i_button.click()

# 等待下拉框出现并点击"Export"
export_button = WebDriverWait(driver, 10).until(EC.element_to_be_clickable((By.XPATH, "//div[@data-context-menu-label='Export']")))
export_button.click()

# 等待导出选项出现并选择"CSV"
csv_button = WebDriverWait(driver, 10).until(EC.element_to_be_clickable((By.XPATH, "//div[contains(text(), 'CSV')]")))
csv_button.click()

# 假设文件已经下载，接下来的步骤是将下载的CSV文件导入数据库，这部分代码将根据实际情况编写

# 注意：实际使用时请根据页面元素进行适当的调整
