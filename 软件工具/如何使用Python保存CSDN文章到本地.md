
> 本文首发地址 <https://h89.cn/archives/223.html>  
> 最新更新地址 <https://gitee.com/chenjim/chenjimblog>  


在CSDN上有许多技术文档写的比较详细，如果想保存下来，需要怎么做呢？  
python 是一把利器，可以对网页读取、解析、保存我们需要的内容。下面就介绍如何使用 python 保存 CSDN 文章到本地。   

1. 安装 python ，参见 <https://www.python.org/downloads/>
2. 安装需要的依赖 `pip install html2text lxml requests`
3. 使用 python 具体实现如下  
    ```python
    # 以下代码源自 https://blog.csdn.net/m0_53268714/article/details/121058706
    import requests
    from html2text import HTML2Text
    from lxml import etree
    from html import unescape
    import os
    import re

    def crawl(url):
        headers = {
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/95.0.4638.54 Safari/537.36",
        }
        # 配置 header 破反爬
        response = requests.get(url, headers=headers)
        # 200 就继续
        if response.status_code == 200:
            html = response.content.decode("utf8")
            # print(html)
            tree = etree.HTML(html)
            # print("look for text...")
            # 找到需要的html块
            title = tree.xpath('//*[@id="articleContentId"]/text()')[0]
            block = tree.xpath('//*[@id="content_views"]')
            # html
            ohtml = unescape(etree.tostring(block[0]).decode("utf8"))
            # 纯文本
            text = block[0].xpath('string(.)').strip()
            # print("html:", ohtml)
            # print("text:", text)

            # 提出特殊字符，比如 https://blog.csdn.net/shulianghan/article/details/115458625 标题存在 "|"
            # 参考自 https://blog.csdn.net/vocalsir/article/details/121993693
            title = re.sub(r'[:/\\?*“”<>|]', '-', title).replace(" ", "")

            print("title:", title)
            save(ohtml, text, title)
            print("finish!")
        else:
            print("failed!")

    def save(html, text, title):
        if "output" not in os.listdir():
            # 不存在输出文件夹就创建
            os.mkdir("output")
        with open(f"output/{title}.html", 'w', encoding='utf8') as html_file:
            # 保存html，可以浏览器打开阅读
            print("write html...")
            # 写入原文信息
            html_file.write("原文  <a href="+url+">"+title+"</a><br>\n")
            html_file.write(html)
            html_file.close()
        with open(f"output/{title}.md", 'w', encoding='utf8') as md_file:
            # 保存markdown，需要 markdown 工具预览
            print("write markdown...")
            # 写入原文信息
            md_file.write("**原文** ["+title+"]("+url+")  \n\n")
            text_maker = HTML2Text()
            # md转换
            md_text = text_maker.handle(html)
            md_file.write(md_text)
            md_file.close()

    if __name__ == '__main__':
        # 你想要爬取的文章url
        url = "https://blog.csdn.net/freekiteyu/article/details/79483406"
        crawl(url
    ```
4. 只需要修改如上`url`为我们需要保存的链接即可  
   CSDN图片有防盗链，如果转载需要更新图片  