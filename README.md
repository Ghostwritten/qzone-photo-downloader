@[toc]

这是我fork 关于[dslwind/qzone-photo-downloader](https://github.com/dslwind/qzone-photo-downloader)的小工具：QQ 空间照片下载器项目，但由于时间长久，下载已经失效。因此我对此做了改进修补工作，这是我的[QQ 空间照片下载器demo](https://github.com/ghostwriten/qzone-photo-downloader)。但你还以直接用更好功能更全的关于qq空间的[wwwpf/QzoneExporter](https://github.com/wwwpf/QzoneExporter)项目。

## 1. 前提

 - ubuntu linux 18.04.5
 - Python 3.6.9

当然，其他版本不能保证，你可以尝试

## 2. 安装依赖

- selenium

  `pip install selenium`

 * [chromedriver](http://chromedriver.storage.googleapis.com/index.html)

```bash
$ wget https://chromedriver.storage.googleapis.com/99.0.4844.51/chromedriver_linux64.zip  
$ unzip chromedriver_linux64.zip
$ apt install libnss3-dev  chromium-bsu
```

> 注意：`chrome-brower`与`chromedirver` 要保持一致。根本apt源安装`chrome-brower`默认的版本是`99`...我选择了chromedirver版本也是`99.0.4844.51`

## 3. 使用方法

### 3.1 克隆
  ```shell
  git clone https://github.com/ghostwritten/qzone-photo-downloader.git
  cd qzone-photo-downloader
  ```

###  3.2 修改qq
  打开`downloader.py`，定位到以下代码

  ```python
  def entry():
      # 你的 QQ和密码，QQ号必须写，密码可以省略，然后使用网页快速登录功能
      main_user = 123456
      main_pass = ''

      # 要处理的目标 QQ 号，此处可填入多个QQ号，中间用逗号隔开
      dest_users = [123456, ]
  ```

```bash
$ python3 downloader.py 
```

## 4 问题

###  4.1 解析错误
```bash
$ python3 downloader.py 
登录成功
登录成功
正在处理用户 xxxx
获取到 0 个相册
未找到 xxxxx 可下载的相册
处理完成
```
我明明有几十个相册，为什么获取不到呢？
在打印变量c结构体的时候，数据解析结构已经发生变化。
这是旧 的获取相册的方式
```bash
    if c:
        c = json.loads(c)
        if ('data' in c) and ('albumListModeSort' in c['data']):
            for i in c['data']['albumListModeSort']:
                albums.append(
                    QzoneAlbum._make([i['id'], i['name'], i['total']]))
    # print(albums)
    return 
```
这是新的方式，多个一层相册分类的属性：

```yaml
        if c:
            c = json.loads(c)
            if ('data' in c) and ('albumListModeClass' in c['data']):
                for i in c['data']['albumListModeClass']:
                  for a in i['albumList']:
                     albums.append(
                     QzoneAlbum._make([a['id'], a['name'], a['total']]))
        #print(albums)
        return albums
```

###  4.2. valid json 问题

 - json.decoder.JSONDecodeError: Expecting ',' delimiter: line 15135
   column 177 (char 527917)
 - json.decoder.JSONDecodeError: Invalid \escape: line 1 column 1 (char
   1)

一般是JSON 中出现特殊字符易出现该 BUG
你可以尝试

```bash
s = json_str.replace('\\', '\\\\')
db = json.loads(s)
```
或者

```bash
c = json.loads(c,strict=False, encoding="utf-8")
```
或者
根据我的相册内容，而我是将json数据打印出来，判断分析是相册描述是特殊最多的一部分，我暂时将数据输入到一个文件

```bash
.....
write_to_json('./data_photo.txt', c)
....
def write_to_json(filename, data):
    jsonfile = open(filename, "w")
    jsonfile.write(data)
...
```
这样会定位特殊字符的位置，根据相片url，我定位到相册位置，删除照片，就跳过该`bug`，或者修改相册描述的字段内容。
```bash
$ cat data_photo.txt_2 | jq .
parse error: Invalid escape at line 26467, column 384

cat -n data_photo.txt_2 |grep 26467
 26467	         "desc" : "人：越来越不男不女了；钱：越来越不干不净了；友：越来越不好不坏了；情：越来越不咸不淡了；义：越来越不轻不重了；官：越来越不清不白了；理：越来越不清不楚了；心：越来越不红不黒\了；话：越来越不冷不热了；爱：越来越不死不活了；路：越来越不明不暗了。",

```
