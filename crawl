import re
import json
from lxml import etree
import requests
import click
from urllib import parse
import time
import pickle
from openpyxl import Workbook

PAT = re.compile(r'queryId:"(\d*)?"', re.MULTILINE)
headers = {
    "Origin": "https://www.instagram.com/",
    "Referer": "https://www.instagram.com/ahmad_monk/",
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) "
                  "Chrome/58.0.3029.110 Safari/537.36",
    "Host": "www.instagram.com",
    "accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",
    "accept-encoding": "gzip, deflate, sdch, br",
    "accept-language": "zh-CN,zh;q=0.8",
    "X-Instragram-AJAX": "1",
    "X-Requested-With": "XMLHttpRequest",
    "Upgrade-Insecure-Requests": "1",
}


jso = {"id": "", "first": 12, "after": ""}

BASE_URL = "https://www.instagram.com"

# QUERY = "/morisakitomomi/"  # 森咲智美
QUERY = "/ahmad_monk/"
NEXT_URL = 'https://www.instagram.com/graphql/query/?query_id={0}&variables={1}'

proxy = {
    'http': 'http://127.0.0.1:1080',
    'https': 'http://127.0.0.1:1080'
}





def crawl():
    click.echo('start...')
    try:
        all_imgs_url = []
        res = requests.get(BASE_URL + QUERY, headers=headers, proxies=proxy)  # , verify=False)
        html = etree.HTML(res.content.decode())
        all_a_tags = html.xpath('//script[@type="text/javascript"]/text()')  # 图片数据源
        query_id_url = html.xpath('//script[@crossorigin="anonymous"]/@src')  # query_id 作为内容加载
        click.echo(query_id_url)
        for a_tag in all_a_tags:
            if a_tag.strip().startswith('window'):
                data = a_tag.split('= {')[1][:-1]  # 获取json数据块
                js_data = json.loads('{' + data, encoding='utf-8')
                # 第一次访问首页获取的image_info
                nodes = js_data["entry_data"]["ProfilePage"][0]["user"]["media"]["nodes"]
                for node in nodes:
                    # print(node)
                    comment_num = node["comments"]["count"]
                    like_num = node["likes"]["count"]
                    if node["display_src"] not in info_finish:
                        click.echo('图片地址：{},  评论数：{}，  点赞数：{}'.format(node["display_src"], comment_num, like_num))
                        file_.write('图片地址：{},  评论数：{}，  点赞数：{} \n'.format(node["display_src"], comment_num, like_num))
                    info_finish.add(node["display_src"])
                click.echo('ok')
                nodes = js_data["entry_data"]["ProfilePage"][0]["user"]["media"]["nodes"]
                end_cursor = js_data["entry_data"]["ProfilePage"][0]["user"]["media"]["page_info"]["end_cursor"]
                has_next = js_data["entry_data"]["ProfilePage"][0]["user"]["media"]["page_info"]["has_next_page"]
                id = nodes[0]["owner"]["id"]
                for node in nodes:
                    # click.echo(node["display_src"])
                    all_imgs_url.append(node["display_src"])
                    # click.echo(end_cursor)

                    # 请求query_id
                    query_content = requests.get(BASE_URL + query_id_url[-2], headers=headers, proxies=proxy)
                    query_id_list = PAT.findall(query_content.text)
                    # for u in query_id_list:
                    #     click.echo(u)  # 查看query_id
                    query_id = query_id_list[-2]
                    count = 0
                    # 更多的图片加载
                    while has_next and count <= 1:
                        jso["id"] = id
                        jso["after"] = end_cursor
                        text = json.dumps(jso)
                        url = NEXT_URL.format(query_id, parse.quote(text))
                        res = requests.get(url, headers=headers, proxies=proxy)
                        time.sleep(2)
                        html = json.loads(res.content.decode(), encoding='utf-8')
                        has_next = html["data"]["user"]["edge_owner_to_timeline_media"]["page_info"]["has_next_page"]
                        end_cursor = html["data"]["user"]["edge_owner_to_timeline_media"]["page_info"]["end_cursor"]
                        edges = html["data"]["user"]["edge_owner_to_timeline_media"]["edges"]
                        for edge in edges:
                            if edge["node"]["display_url"] not in info_finish:
                                click.echo("图片地址：{}   评论数：{}   点赞数：{} \n".format(edge["node"]["display_url"],edge["node"]["edge_media_to_comment"]["count"],edge["node"]["edge_media_preview_like"]["count"]))
                                file_.write("图片地址：{}   评论数：{}   点赞数：{} \n".format(edge["node"]["display_url"],edge["node"]["edge_media_to_comment"]["count"],edge["node"]["edge_media_preview_like"]["count"]))
                            info_finish.add(node["display_src"])
                            count += 1
                            # all_imgs_url.append(edge["node"]["display_url"])
                    click.echo('ok')
                    with open('info.txt','wb') as e:
                        e.write(pickle.dumps(info_finish))
    except Exception as e:
        raise e

if __name__ == '__main__':
    file_ = open('image.csv', 'a',encoding='utf-8')  # 这个是保存 数据
    try:
        with open('info.txt', 'rb')as f:
            info_finish = pickle.load(f)  #用pickle记录爬取状态
            click.echo('正在检查更新数据')
    except:
        info_finish = set()
    crawl()

#  ps: 只利用了简单的set去重，做增量
