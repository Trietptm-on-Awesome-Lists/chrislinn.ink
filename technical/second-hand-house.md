# 杭州链家二手房爬虫

+ https://github.com/XuefengHuang/lianjia-scrawler
+ https://github.com/jumper2014/lianjia-beike-spider

## Prerequisites
```
git clone https://github.com/ChrisLinn/lianjia-scrawler.git
cd lianjia-scrawler # If you'd like not to use [virtualenv](https://virtualenv.pypa.io/en/stable/), please skip step 3 and 4.
virtualenv lianjia
source lianjia/bin/activate
sudo apt-get install libmysqlclient-dev
pip install -r requirements.txt
python scrawl.py 
```

## CREATE DATABASE `ershoufang` DEFAULT CHARACTER SET utf8;

## Settings.py
```
CITY = 'hz'  # only one, shanghai=sh shenzhen=sh......
REGIONLIST = [u'xihu', u'xiacheng', u'jianggan', u'gongshu', u'shangcheng', u'binjiang']  # only pinyin support
```


## Todo
+ 先爬。
+ 以后重新跑呢？16 行？first time？
    * 历史信息
+ GetHouseByRegionlist 100 pages?
+ 小区信息
+ 可视化
    * https://github.com/XuefengHuang/lianjia-scrawler/blob/master/data/lianjia.ipynb