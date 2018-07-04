#python下的多线程和多进程爬虫-结合队列实现

date: 2018-03-08 22:08:48
tags: [爬虫,多线程]


## 单线程爬虫
单线程的爬虫速度太慢，对应的我们可以使用多线程或者是进程版本来实现
举个例子，抓取糗事百科热门栏目下的十三个url地址的段子内容，地址: https://www.qiushibaike.com/
<!--more-->
普通面向对象版本,非常简单，没有什么值得说的内容

```python
# coding=utf-8
import requests
from lxml import etree

class QiubaiSpider:
    def __init__(self):
        self.url_temp = "https://www.qiushibaike.com/8hr/page/{}/"
        self.headers = {"User-Agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X \
        10_13_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.186 Safari/537.36"}
    def get_url_list(self): #获取url列表
        return [self.url_temp.format(i) for i in range(1,14)]

    def parse_url(self,url): #发送请求，获取响应
        print(url)
        return requests.get(url,headers=self.headers).content.decode()

    def get_content_list(self,html_str): #提取段子
        html = etree.HTML(html_str)
        div_list = html.xpath("//div[@id='content-left']/div")
        content_list = []
        for div in div_list:
            content = {}
            content["content"]=div.xpath(".//div[@class='content']/span/text()")
            content_list.append(content)
        return content_list

    def save_content_list(self,content_list): # 保存数据
        pass

    def run(self):
        #1. url_list
        url_list = self.get_url_list()
        #2. 遍历，发送请求
        for url in url_list:
            html_str = self.parse_url(url)
            #3. 提取数据
            content_list = self.get_content_list(html_str)
            #4. 保存
            self.save_content_list(content_list)

if __name__ == '__main__':
    qiubai = QiubaiSpider()
    qiubai.run()
```

## 多线程爬虫
但是类似的单线程程序太慢，对应的可以考虑多线程实现，四个函数使用多个线程实现，分别使用三个队列存放数据  
代码实现如下：

```python
# coding=utf-8
import requests
from lxml import etree
from queue import Queue
import threading


class Qiubai:
    def __init__(self):
        self.temp_url = "https://www.qiushibaike.com/8hr/page/{}/"
        self.headers= {"User-Agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X \
        10_13_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.186 Safari/537.36"}
        self.url_queue = Queue()
        self.html_queue = Queue()
        self.content_list_queue = Queue()

    def get_url_list(self):#获取url列表
        for i in range(1,14):
            self.url_queue.put(self.temp_url.format(i))

    def parse_url(self):
        while True: #在这里使用，子线程不会结束，把子线程设置为守护线程
            url = self.url_queue.get()
            print(url)
            response = requests.get(url,headers=self.headers)
            self.html_queue.put(response.content.decode())
            self.url_queue.task_done()


    def get_content_list(self):  #提取数据
        while True:
            html_str = self.html_queue.get()
            html = etree.HTML(html_str)
            div_list = html.xpath("//div[@id='content-left']/div")
            content_list = []
            for div in div_list:
                content = {}
                content["content"] = div.xpath(".//div[@class='content']/span/text()")
                content_list.append(content)
            self.content_list_queue.put(content_list)
            self.html_queue.task_done()

    def save_content_list(self):
        while True:
            content_list = self.content_list_queue.get()
            pass
            self.content_list_queue.task_done()

    def run(self):
        thread_list = []
        #1.url_list
        t_url = threading.Thread(target=self.get_url_list)
        thread_list.append(t_url)
        #2.遍历，发送请求，
        for i in range(3):  #三个线程发送请求
            t_parse = threading.Thread(target=self.parse_url)
            thread_list.append(t_parse)
        #3.提取数据
        t_content = threading.Thread(target=self.get_content_list)
        thread_list.append(t_content)
        #4.保存
        t_save = threading.Thread(target=self.save_content_list)
        thread_list.append(t_save)

        for t in thread_list:
            t.setDaemon(True)  #把子线程设置为守护线程，当前这个线程不重要，主线程结束，子线程技术
            t.start()

        for q in [self.url_queue,self.html_queue,self.content_list_queue]:
            q.join()  #让主线程阻塞，等待队列的计数为0，

        print("主线程结束")

if __name__ == '__main__':
    qiubai = Qiubai()
    qiubai.run()
```
上述代码中，put会让队列的计数+1，但是单纯的使用get不会让其-1，需要和task_done同时使用才能够-1；同时task_done不能放在另一个队列的put之前，否则可能会出现数据没有处理完成，程序结束的情况

## 多进程爬虫
这种方式由于GIL全局锁的存在，多线程在python3下可能只是个摆设，对应的解释器执行其中的内容的时候仅仅是顺序执行，此时我们可以考虑多进程的方式实现，思路和多线程相似，只是对应的api不相同。
具体的实现如下:

```python
# coding=utf-8
import requests
from lxml import etree
import threading
from multiprocessing import Process
from multiprocessing import JoinableQueue as Queue

class QiubaiSpider:
    def __init__(self):
        self.url_temp = "https://www.qiushibaike.com/8hr/page/{}/"
        self.headers = {"User-Agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.186 Safari/537.36"}
        self.url_queue = Queue()  #保存url
        self.html_queue = Queue() #保存html字符串
        self.content_queue = Queue() #保存提取到的数据

    def get_url_list(self):
        for i in range(1,14):
            self.url_queue.put(self.url_temp.format(i))

    def parse_url(self):
        while True:
            url = self.url_queue.get()
            print(url)
            html_str =  requests.get(url,headers=self.headers).content.decode()
            self.html_queue.put(html_str)
            self.url_queue.task_done()

    def get_content_list(self):
        while True:
            html_str = self.html_queue.get()
            html = etree.HTML(html_str)
            div_list = html.xpath("//div[@id='content-left']/div")
            content_list = []
            for div in div_list:
                content = {}
                content["content"]=div.xpath(".//div[@class='content']/span/text()")
                content_list.append(content)
            self.content_queue.put(content_list)
            self.html_queue.task_done()

    def save_content_list(self):
        while True:
            content_list = self.content_queue.get()
            pass
            self.content_queue.task_done()

    def run(self):
        process_list = []
        #1. url_list
        t_url = Process(target=self.get_url_list)
        process_list.append(t_url)
        #2. 遍历，发送请求
        for i in range(5):#创建5个子进程
            t_parse = Process(target=self.parse_url)
            process_list.append(t_parse)
        #3. 提取数据
        t_content = Process(target=self.get_content_list)
        process_list.append(t_content)
        #4. 保存
        t_save = Process(target=self.save_content_list)
        process_list.append(t_save)

        for t in process_list:
            t.daemon=True #把进线程设置为守护线程，主进程技术，子进程结束
            t.start()

        for q in [self.url_queue,self.html_queue,self.content_queue]:
            q.join()  #让主进程阻塞

        print("主进程结束")

if __name__ == '__main__':
    qiubai = QiubaiSpider()
    qiubai.run()
```
上述多进程实现的代码中，multiprocessing提供的JoinableQueue可以创建可连接的共享进程队列。和普通的Queue对象一样，队列允许项目的使用者通知生产者项目已经被成功处理。通知进程是使用共享的信号和条件变量来实现的。 对应的该队列能够和普通队列一样能够调用task_done和join方法
