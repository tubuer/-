# -import requests
import jsonpath
import time
from lxml import etree
import re
import json


def get_title(sku):
    url = f'https://item.jd.com/{sku}.html'
    # session 会话保持
    s = requests.session()
    s.headers = {
        'accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
        'accept-encoding': 'gzip, deflate, br',
        'accept-language': 'zh-CN,zh;q=0.9',
        'cache-control': 'no-cache',
        'cookie': 'unpl=V2_ZzNtbUpRQkd0ChFVex1YUWICG1URABBCcV1AB30dX1ZlVhVcclRCFnUUR1xnGV4UZgAZWEVcRhdFCEdkeBBVAWMDE1VGZxBFLV0CFSNGF1wjU00zQwBBQHcJFF0uSgwDYgcaDhFTQEJ2XBVQL0oMDDdRFAhyZ0AVRQhHZHMbVQJgBhFbRWdzEkU4dld9H1QHYjMTbUNnAUEpCk5WeRhfSG8BG1pFUkATcjhHZHg%3d; __jdv=76161171|baidu-pinzhuan|t_288551095_baidupinzhuan|cpc|0f3d30c8dba7459bb52f2eb5eba8ac7d_0_970c03f0155e4089bfbf5d7b752b3d60|1631472401841; __jdu=938664276; areaId=15; ipLoc-djd=15-1213-1214-0; PCSYCityID=CN_330000_330100_330110; shshshfpa=3f241791-55d5-a010-29fa-c43074f60e95-1631472404; shshshfp=bc4d8cdea65a9e29c181a5635c2c3ac1; shshshfpb=d7h55ii%2Fd9a57c%20vYcdWxoA%3D%3D; __jdc=122270672; __jda=122270672.938664276.1631472401.1631472402.1631565014.2; token=941911f63c905fa9386a397b6d020318,2,906425; __tk=4uj5BUa1AUWEBVXrjzXtAcaz4znxAugwBYbt4YW1Arn,2,906425; shshshsID=fa1e431b4c1a6c3c073ccf0ac259903d_2_1631565016807; __jdb=122270672.2.938664276|2.1631565014; 3AB9D23F7A4B3C9B=N5EDSAZ6YF6TKJ7BG2U3WSPHQIZWPKZW6QZPOXNBDGWY7ZOXWDESLPSS7QRB5INGYP6ICBOH33XHT3FZPBIK2KAIMM',
        'pragma': 'no-cache',
        'referer': 'https://search.jd.com/',
        'sec-ch-ua': '"Chromium";v="92", " Not A;Brand";v="99", "Google Chrome";v="92"',
        'sec-ch-ua-mobile': '?0',
        'sec-fetch-dest': 'document',
        'sec-fetch-mode': 'navigate',
        'sec-fetch-site': 'same-site',
        'sec-fetch-user': '?1',
        'upgrade-insecure-requests': '1',
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.159 Safari/537.36'
    }
    # 通过会话对象发送网络请求
    resp = s.get(url)
    text = resp.text
    html = etree.HTML(text)
    title = html.xpath('/html/head/title/text()')[0].strip().replace(',', '.').replace('，', '.')

    # 获取商品介绍
    params = html.xpath('/html/body/div[9]/div[2]/div[1]/div[2]/div[1]/div[1]/ul[2]/li')

    # 设置默认的规格为 ’*‘
    param_text = '*'
    for k in params:
        param_text_temp = k.xpath('./text()')

        # 1.长度大于1 ， 2. 数据里包含’规格‘
        if (len(param_text_temp) > 0 and param_text_temp[0].find('型号') != -1):
            param_text = param_text_temp[0]
    return title, param_text


# 获取评论信息    评论数量，好评度，好评数
def get_comment(sku):
    # 详情页抓包
    for l in range(0, 1):
        comment_url = f'https://club.jd.com/comment/productPageComments.action?callback=fetchJSON_comment98&productId={sku}&score=0&sortType=5&page={l}&pageSize=10&isShadowSku=0&fold=1'
        s = requests.session()
        s.headers = {
            'accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
            'accept-encoding': 'gzip, deflate, br',
            'accept-language': 'zh-CN,zh;q=0.9',
            'cache-control': 'no-cache',
            'cookie': 'unpl=V2_ZzNtbUpRQkd0ChFVex1YUWICG1URABBCcV1AB30dX1ZlVhVcclRCFnUUR1xnGV4UZgAZWEVcRhdFCEdkeBBVAWMDE1VGZxBFLV0CFSNGF1wjU00zQwBBQHcJFF0uSgwDYgcaDhFTQEJ2XBVQL0oMDDdRFAhyZ0AVRQhHZHMbVQJgBhFbRWdzEkU4dld9H1QHYjMTbUNnAUEpCk5WeRhfSG8BG1pFUkATcjhHZHg%3d; __jdv=76161171|baidu-pinzhuan|t_288551095_baidupinzhuan|cpc|0f3d30c8dba7459bb52f2eb5eba8ac7d_0_970c03f0155e4089bfbf5d7b752b3d60|1631472401841; __jdu=938664276; areaId=15; ipLoc-djd=15-1213-1214-0; PCSYCityID=CN_330000_330100_330110; shshshfpa=3f241791-55d5-a010-29fa-c43074f60e95-1631472404; shshshfp=bc4d8cdea65a9e29c181a5635c2c3ac1; shshshfpb=d7h55ii%2Fd9a57c%20vYcdWxoA%3D%3D; __jdc=122270672; __jda=122270672.938664276.1631472401.1631472402.1631565014.2; token=941911f63c905fa9386a397b6d020318,2,906425; __tk=4uj5BUa1AUWEBVXrjzXtAcaz4znxAugwBYbt4YW1Arn,2,906425; shshshsID=fa1e431b4c1a6c3c073ccf0ac259903d_2_1631565016807; __jdb=122270672.2.938664276|2.1631565014; 3AB9D23F7A4B3C9B=N5EDSAZ6YF6TKJ7BG2U3WSPHQIZWPKZW6QZPOXNBDGWY7ZOXWDESLPSS7QRB5INGYP6ICBOH33XHT3FZPBIK2KAIMM',
            'pragma': 'no-cache',
            'referer': 'https://search.jd.com/',
            'sec-ch-ua': '"Chromium";v="92", " Not A;Brand";v="99", "Google Chrome";v="92"',
            'sec-ch-ua-mobile': '?0',
            'sec-fetch-dest': 'document',
            'sec-fetch-mode': 'navigate',
            'sec-fetch-site': 'same-site',
            'sec-fetch-user': '?1',
            'upgrade-insecure-requests': '1',
            'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.159 Safari/537.36'
        }
        resp = s.get(comment_url)
        text = resp.text
        text1 = re.sub(u'^fetchJSON_comment98\(', '', text)
        text2 = re.sub(u'\);', '', text1)
        data_ = json.loads(text2)
        print(data_)
        content_list = jsonpath.jsonpath(data_, '$..content')
        nickname_list = jsonpath.jsonpath(data_, '$..nickname')
        print(comment_url)
        if nickname_list == False:
            r = ['w', 'p']
        else:
            for i in range(len(nickname_list)):
                content = content_list[i]
                nickname = nickname_list[i]
                r = [nickname, content]

    return r


# 获取价格
def get_price(sku):
    p_url = f'https://p.3.cn/prices/mgets?skuIds=J_{sku}'
    s = requests.session()
    s.headers = {
        'accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
        'accept-encoding': 'gzip, deflate, br',
        'accept-language': 'zh-CN,zh;q=0.9',
        'cache-control': 'no-cache',
        'cookie': 'unpl=V2_ZzNtbUpRQkd0ChFVex1YUWICG1URABBCcV1AB30dX1ZlVhVcclRCFnUUR1xnGV4UZgAZWEVcRhdFCEdkeBBVAWMDE1VGZxBFLV0CFSNGF1wjU00zQwBBQHcJFF0uSgwDYgcaDhFTQEJ2XBVQL0oMDDdRFAhyZ0AVRQhHZHMbVQJgBhFbRWdzEkU4dld9H1QHYjMTbUNnAUEpCk5WeRhfSG8BG1pFUkATcjhHZHg%3d; __jdv=76161171|baidu-pinzhuan|t_288551095_baidupinzhuan|cpc|0f3d30c8dba7459bb52f2eb5eba8ac7d_0_970c03f0155e4089bfbf5d7b752b3d60|1631472401841; __jdu=938664276; areaId=15; ipLoc-djd=15-1213-1214-0; PCSYCityID=CN_330000_330100_330110; shshshfpa=3f241791-55d5-a010-29fa-c43074f60e95-1631472404; shshshfp=bc4d8cdea65a9e29c181a5635c2c3ac1; shshshfpb=d7h55ii%2Fd9a57c%20vYcdWxoA%3D%3D; __jdc=122270672; __jda=122270672.938664276.1631472401.1631472402.1631565014.2; token=941911f63c905fa9386a397b6d020318,2,906425; __tk=4uj5BUa1AUWEBVXrjzXtAcaz4znxAugwBYbt4YW1Arn,2,906425; shshshsID=fa1e431b4c1a6c3c073ccf0ac259903d_2_1631565016807; __jdb=122270672.2.938664276|2.1631565014; 3AB9D23F7A4B3C9B=N5EDSAZ6YF6TKJ7BG2U3WSPHQIZWPKZW6QZPOXNBDGWY7ZOXWDESLPSS7QRB5INGYP6ICBOH33XHT3FZPBIK2KAIMM',
        'pragma': 'no-cache',
        'referer': 'https://search.jd.com/',
        'sec-ch-ua': '"Chromium";v="92", " Not A;Brand";v="99", "Google Chrome";v="92"',
        'sec-ch-ua-mobile': '?0',
        'sec-fetch-dest': 'document',
        'sec-fetch-mode': 'navigate',
        'sec-fetch-site': 'same-site',
        'sec-fetch-user': '?1',
        'upgrade-insecure-requests': '1',
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.159 Safari/537.36'
    }
    resp = s.get(p_url)
    text = json.loads(resp.text)
    price = jsonpath.jsonpath(text, '$..p')[0]
    return price


keyword = '3080显卡'
for i in range(1, 6):
    if i % 2 != 0:

        url = f"https://search.jd.com/Search?keyword={keyword}&wq{keyword}&pvid=d5a92f53d1444e659c472c660d403051&page={i}&s=1&click=1"

        headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:89.0) Gecko/20100101 Firefox/89.0'
        }

        resp = requests.get(url, headers=headers)
        text = resp.text
        # 转化成xp能够处理的数据
        html = etree.HTML(text)

        # 拿到商品列表也所有的商品信息
        sku_list = html.xpath('//*[@id="J_goodsList"]/ul/li')
        for ii in sku_list:
            sku = ii.xpath('./@data-sku')[0]
            print(keyword, sku)
            # 保存数据
            with open('sku_list.csv', 'a') as f:
                f.write(f'{keyword},{sku}\n')
        print(url)
    else:
        continue

"""
读取文件，对商品详情页进行获取
"""
with open('sku_list.csv', 'r', encoding='gb18030') as f:

    file = f.readlines()

# 建立空列表，多sku进行去重
ed_list = []

# 循环读取csv每行数据
for i in file:
    keyword, sku = i.split(',')
    # 截取sku字符串， 去掉 ‘\n’
    sku = sku[:-1]

    # 去重
    if sku in ed_list:
        continue
    else:
        ed_list.append(sku)

    # 获取商品标题
    title, param_text = get_title(sku)

    # 获取评论数据
    r_list = get_comment(sku)
    r_nane = r_list[0]
    content = r_list[1]

    # 获取价格
    price = get_price(sku)

    info = [keyword, sku, title, price, r_nane, content]
    with open('JD.csv', 'a', encoding='gb18030') as f:
        f.write(f"{keyword},{sku},{title},{price},{r_nane},{content}\n")
    print(info)
    time.sleep(1)
print(1)
