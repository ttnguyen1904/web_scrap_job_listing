# web_scrap_job_listing
Web scraping for job listing in Vietnam targeting details such as title, location, industry over the period of 1 months for further analysis on labour market demographic 

""""""""""""""""""""""""""""""""""
SCRAPPING WEBSITE
Case: Vietnamwork
Target data: Posted Job detail

"""""""""""""""""""""""""""
import sys
!{sys.executable} -m pip install -U selenium
from selenium import webdriver
from selenium.webdriver.support.ui import Select
from selenium.webdriver.common.keys import Keys
from selenium.common.exceptions import NoSuchElementException

import time

# os provides a portable way of using operating system dependent functionality.
import os
os.environ["PATH"] += os.pathsep + r'C:\Users\Hoang Ly\chromedriver'

# open the website through Chrome using webdriver
driver = webdriver.Chrome()
driver.get('https://www.vietnamworks.com/tim-viec-lam/tat-ca-viec-lam')
driver.implicitly_wait(30)
time.sleep(10)

# PROBLEM WHEN ENTER USERNAME AND PASSWORD. WAS ABLE TO OPEN THE 'SIGN IN' BUTTON. NEED MORE WORK
# driver.find_element_by_xpath("//li/a[@class='clickable']").click()

# find all job links in vietnamwork
"""""""""""""""""""""" 
links = set()

for _ in range(215):
    try:
        link = driver.find_elements_by_xpath("//h3/a[@href]")
        for x in link:
            text = x.get_attribute("href")
            links.add(text)
        driver.find_element_by_xpath("//li[@class='ais-pagination--item ais-pagination--item__next']/a[@class='ais-pagination--link']").click() 
        time.sleep(30)
    
    except NoSuchElementException:  # the next button changes from page 20 onward for _ in range(195)
        link = driver.find_elements_by_xpath("//h3/a[@href]")
        for x in link:
            text = x.get_attribute("href")
            links.add(text)
        driver.find_element_by_xpath("//li/span[@class='job-search__load-more-jobs']/span[@class='clickable']").click() 
        time.sleep(30)
        
"""""""""""""""""""""" 

# Handling interactive website
main_window = driver.current_window_handle

# create a list of all required information 
title = []
salary = []
level = []
indus = []
location = []
company_name = []

# loop for each job links
for link in [x for x in links][:9532]: #chose to which page to stop
    driver.execute_script('''window.open('%s',"_blank");'''%link)   #open a new tap with job link
    window_after = driver.window_handles[1]        #move forcus of action to new tap. Old tap has index of 0
    driver.switch_to_window(window_after)
    time.sleep(5)
    try:
        title_t = driver.find_element_by_xpath("//div[@class='job-header-info']/h1[@class='job-title']")
        salary_t = driver.find_element_by_xpath("//span[@class='salary']/strong[@class='text-primary text-lg']") 
        company = driver.find_elements_by_xpath("//div[@class='col-sm-12 company-name']/a[@class='track-event']")
        summary = driver.find_elements_by_xpath("//div[@class='col-xs-10 summary-content']/span[@class='content']")
        locations = driver.find_elements_by_xpath("//span[@class='company-location']/a[@itemprop='address']")
    
        title.append(title_t.text)    
        salary.append(salary_t.text)    
        level.append(summary[1].text)
        indus.append(summary[2].text)
    
        location_t = ""
        for x in range(10):
            try:
                loc_x = locations[x].text
                location_t = location_t + loc_x + ","
            except IndexError:
                loc_x = 'null'
        location_t = location_t[:-1]
        location.append(location_t)
    
        company_t = ""
        for x in range(10):
            try:
                company_x = company[x].text
                if company_x != "":
                    company_t = company_x
            except IndexError:
                company_x = 'null'
        company_name .append(company_t) 
    
        driver.close()
        driver.switch_to_window(main_window)
        except NoSuchElementException:         #for links that are not working, we close the tab, switch forcus to main tab and start the loop again
        title_t = 'null' 
        driver.close()
        driver.switch_to_window(main_window)       


# Since the data is in Vietnamese. We export it in excel
import xlsxwriter

workbook = xlsxwriter.Workbook('C:\\Users\\Hoang Ly\\Desktop\\Python\\Vietnamwork job posting_20180835.xlsx')
worksheet = workbook.add_worksheet()

# Add a bold format to use to highlight cells.
bold = workbook.add_format({'bold': True})

# Write some data headers in bold font.
worksheet.write('A1', 'Company name', bold)
worksheet.write('B1', 'Industry', bold)
worksheet.write('C1', 'Job Location', bold)
worksheet.write('D1', 'Job Title', bold)
worksheet.write('E1', 'Job Salary', bold)
worksheet.write('F1', 'Level', bold)

# Start from the first cell below the header. Rows and columns are zero indexed.
row = 1
col = 0 

for (a, b, c, d, e, f) in zip(company_name, indus, location, title, salary, level):
    worksheet.write(row, col,   a )
    worksheet.write(row, col+1, b )
    worksheet.write(row, col+2, c )
    worksheet.write(row, col+3, d )
    worksheet.write(row, col+4, e )
    worksheet.write(row, col+5, f )
    row += 1
workbook.close()
