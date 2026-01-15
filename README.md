# Robo-automatizar-teste-de-ONT
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.chrome.options import Options
import time
import re
import winsound
import sys

#----- INTERFACE MODEM EG8145V5-V2 (5.8) -----#

service = Service(executable_path="chromedriver.exe")
driver = webdriver.Chrome(service=service)
driver.get("http://192.168.18.1/")
driver.maximize_window()

# -----Login da Interface-----#
WebDriverWait(driver, 5).until(EC.presence_of_element_located((By.CLASS_NAME, "logininputcss")))

#-----Usuário-----#
input_element = driver.find_element(By.CLASS_NAME, "logininputcss")
input_element.send_keys("Epadmin")

#-----Senha-----#
input_element = driver.find_element(By.ID, "txt_Password")
input_element.send_keys("adminEp" , Keys.ENTER)
        
#----- Caminho Fibra-----#
WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.ID, "icon_Systeminfo")))
System_id = "icon_Systeminfo"
System = driver.find_element(By.ID, "icon_Systeminfo")
System.click()
driver.execute_script("document.getElementById('name_opticinfo').click()")
time.sleep(4)

#----- Potência -----#

#-- Entra no Iframe --#
driver.switch_to.frame(0)

rx_element = WebDriverWait(driver, 5).until(
    EC.visibility_of_element_located(
        (By.XPATH, "//td[contains(., 'RX Optical')]/following-sibling::td[1]")
    )
)
rx_text = rx_element.text
rx_value = float(rx_text.replace("dBm", "").strip())

print("RX Optical Power:", rx_value)

#----- Validar a Potência -----# 

if not (-26 <= rx_value <= -18):
    raise Exception("Potência RX fora do padrão")

print("A fibra esta boa!!")

driver.switch_to.default_content()

#----- Arquivo -----#

driver.execute_script("document.getElementById('icon_addconfig').click()")
driver.execute_script("document.getElementById('name_maintaininfo').click()")
driver.execute_script("document.getElementById('cfgconfig').click()")
time.sleep(3)

#----- Lançar arquivo -----#
driver.switch_to.frame(0)

wait = WebDriverWait(driver, 2)

inputs = wait.until(
    EC.presence_of_all_elements_located((By.ID, "t_file"))
)
inputs[0].send_keys(r"C:\Users\user\Desktop\HUAWEI 5.8.xml")
driver.execute_script("document.getElementById('btnSubmit').click()")

alert = wait.until(EC.alert_is_present())
alert.accept() 

time.sleep(10)
