---
title: 抓取拉勾网招聘信息
date: 2018-03-23 16:13:46
tags: python
categories: python
---


{% note info %}

	通过查看页面元素,找到`json`数据,这里面有当页全部的信息.直接处理这一部分就可以了
	使用`pandas`将数据写入csv中


{% endnote %}

<!-- more -->


下面是内容:

```python


import requests
import time
import sys
import random
import pandas as pd


# 全国
# url = r'https://www.lagou.com/jobs/positionAjax.json?needAddtionalResult=false&isSchoolJob=0'
# 上海
url = r'https://www.lagou.com/jobs/positionAjax.json?city=%E4%B8%8A%E6%B5%B7&needAddtionalResult=false&isSchoolJob=0'

header = {
    'Accept': "application/json, text/javascript, */*; q=0.01",
    'Accept-Encoding': "gzip, deflate, br",
    'Accept-Language': "zh-CN,zh;q=0.9",
    'Connection': "keep-alive",
    'Content-Length': "25",
    'Content-Type': "application/x-www-form-urlencoded; charset=UTF-8",
    'Cookie': "_ga=GA1.2.148222605.1514211497; user_trace_token=20171225221816-72dec81c-e97e-11e7-9e6c-5254005c3644; LGUID=20171225221816-72decce5-e97e-11e7-9e6c-5254005c3644; JSESSIONID=ABAAABAAAFCAAEG1260B682CC05B68D0702AFC0BE7C9F99; X_HTTP_TOKEN=873379ecae5efb8078bbae146cab2317; showExpriedIndex=1; showExpriedCompanyHome=1; showExpriedMyPublish=1; hasDeliver=5; index_location_city=%E5%85%A8%E5%9B%BD; hideSliderBanner20180305WithTopBannerC=1; _gid=GA1.2.1510422010.1521084881; Hm_lvt_4233e74dff0ae5bd0a3d81c6ccf756e6=1520662114,1520997777,1521084881; gate_login_token=019c2049156a849e6fc5c8bbba46b6b4bfc3f7e1c28e88b7; TG-TRACK-CODE=search_code; login=false; unick=""; gate_login_token=""; _putrc=""; LGRID=20180315120231-af2665e9-2805-11e8-b212-525400f775ce; Hm_lpvt_4233e74dff0ae5bd0a3d81c6ccf756e6=1521086551; SEARCH_ID=3a699c076cea44bb92ebd0bd7db821f0",
    'Host': "www.lagou.com",
    'Origin': "https://www.lagou.com",
    'Referer': "https://www.lagou.com/jobs/list_python?city=%E5%85%A8%E5%9B%BD&cl=false&fromSearch=true&labelWords=&suginput=",
    'User-Agent': "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.162 Safari/537.36",
    'X-Anit-Forge-Code': "0",
    'X-Anit-Forge-Token': "None",
    'X-Requested-With': "XMLHttpRequest"
}

all_info = []
kd = "python"
csv_name = 'sh_ops'


def get_page_num(url, header):
	'''获取招聘页数'''

    form = {
        'first': "true",
        "pn": str(1),
        "kd": kd
    }

    html = requests.post(url, form, headers=header)
    # pprint(html.text)

    page_num = (html.json()['content']['positionResult']['totalCount']) / 15
    # 页面最多显示30个,所以我们限定一下
    if page_num > 30:
        return 30
    else:
        return int(page_num)

for n in range(1, get_page_num(url, header) + 1):

    # 表单信息
    form = {
        'first': "true",
        "pn": str(n),
        "kd": kd
    }

    time.sleep(random.randint(3, 6))

    # 发送post请求，提交表单
    html = requests.post(url, form, headers=header)
    # pprint(html.text)

    # 抓取的职位信息
    position_results = html.json()['content']['positionResult']['result']

    # 这个字段不为True代表数据获取失败
    if html.json()['success'] == True:
        for result in position_results:
            # 遍历单页中单个职位的内容
            info = {}

            info['城市'] = result['city']
            info['区域'] = result['district']
            info['职位名称'] = result['positionName']
            info['工作年限'] = result['workYear']
            info['教育程度'] = result['education']
            info['工作性质'] = result['jobNature']
            info['薪资'] = result['salary']
            info['公司性质'] = result['financeStage']
            info['公司行业'] = result['industryField']
            info['公司规模'] = result['companySize']
            info['公司短名称'] = result['companyShortName']
            info['类型'] = result['firstType']
            info['定位'] = result['secondType']
            info['公司全名'] = result['companyFullName']
            info['URL'] = r'https://www.lagou.com/jobs/{0}.html'.format(result['positionId'])

            all_info.append(info)
        print("第{0}页好了".format(n))

    else:
        print('Error')
        sys.exit(1)

data = pd.DataFrame(all_info)

# 使用pandas,导出到csv文件中
data.to_csv(r'd:\Users\xddd\Documents\GitHub\python_spider\spider_series\{0}.csv'.format(csv_name),
            index=False, mode='w+', encoding='utf_8_sig')

```


效果图
{% asset_img spider_lagou.jpg 这是最终结果%}