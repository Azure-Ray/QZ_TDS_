email_input = WebDriverWait(driver, 10).until(
    EC.presence_of_element_located((By.ID, "i0116"))
)
email_input.send_keys("your_email@example.com" + Keys.ENTER)

# 输入密码
# 注意：Microsoft的登录流程可能涉及到动态加载和额外的验证步骤，这里可能需要调整等待时间或添加额外的处理步骤
password_input = WebDriverWait(driver, 10).until(
    EC.presence_of_element_located((By.ID, "i0118"))
)
password_input.send_keys("your_password" + Keys.ENTER)
