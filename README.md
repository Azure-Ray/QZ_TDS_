import os
import shutil

def clear_download_folder(folder_path):
    for filename in os.listdir(folder_path):
        file_path = os.path.join(folder_path, filename)
        try:
            if os.path.isfile(file_path) or os.path.islink(file_path):
                os.unlink(file_path)
            elif os.path.isdir(file_path):
                shutil.rmtree(file_path)
        except Exception as e:
            print(f'Failed to delete {file_path}. Reason: {e}')


try:
    driver.get("your_login_page_url")
    email_input = WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.ID, "i0116"))
    )
    email_input.send_keys("your_email@example.com" + Keys.ENTER)
    
    # 填写密码和其他登录步骤
    # ...
except Exception as e:
    print(f"Already logged in or login page not loaded properly. Reason: {e}")
