# ProxyPool

![build](https://github.com/Python3WebSpider/ProxyPool/workflows/build/badge.svg)
![deploy](https://github.com/Python3WebSpider/ProxyPool/workflows/deploy/badge.svg)
![](https://img.shields.io/badge/python-3.6%2B-brightgreen)
![Docker Pulls](https://img.shields.io/docker/pulls/germey/proxypool)

Simple and efficient agent pool, providing the following functions:

* Fetch free proxy websites regularly, easy and expandable.
* Use Redis to store agents and sort agent availability.
* Regular testing and screening, remove unavailable agents, and leave available agents.
* Provide proxy API, and randomly select available proxies that have passed the test.

The principle analysis of the proxy pool can be seen in "[How to build an efficient proxy pool](https://cuiqingcai.com/7048.html)", it is recommended to read before using it.

## Run example

API Server can be found in [Deployment Sample](https://proxypool.scrape.center/), random proxy [Access Address](https://proxypool.scrape.center/random), there are fewer proxy sources, only for Demo.

This example is the result of GitHub Actions + Kubernetes automatically deploying the master branch code.

## Requirements

There are two ways to run the agent pool, one way is to use Docker (recommended), and the other way is to run it in a regular way.

### Docker

If you use Docker, you need to install the following environment:

* Docker
* Docker-Compose

### Conventional way

The conventional method requires a Python environment and a Redis environment. The specific requirements are as follows:

* Python>=3.6
* Redis

## Docker run

If Docker and Docker-Compose are installed, you only need one command to run.

```shell script
docker-compose up
```

The results are similar to the following:

```
redis | 1:M 19 Feb 2020 17:09:43.940 * DB loaded from disk: 0.000 seconds
redis | 1:M 19 Feb 2020 17:09:43.940 * Ready to accept connections
proxypool | 2020-02-19 17:09:44,200 CRIT Supervisor is running as root. Privileges were not dropped because no user is specified in the config file. If you intend to run as root, you can set user=root in the config file to avoid this message.
proxypool | 2020-02-19 17:09:44,203 INFO supervisord started with pid 1
proxypool | 2020-02-19 17:09:45,209 INFO spawned:'getter' with pid 10
proxypool | 2020-02-19 17:09:45,212 INFO spawned:'server' with pid 11
proxypool | 2020-02-19 17:09:45,216 INFO spawned:'tester' with pid 12
proxypool | 2020-02-19 17:09:46,596 INFO success: getter entered RUNNING state, process has stayed up for> than 1 seconds (startsecs)
proxypool | 2020-02-19 17:09:46,596 INFO success: server entered RUNNING state, process has stayed up for> than 1 seconds (startsecs)
proxypool | 2020-02-19 17:09:46,596 INFO success: tester entered RUNNING state, process has stayed up for> than 1 seconds (startsecs)
```

You can see that Redis, Getter, Server, and Tester have all started successfully.

At this time, visit [http://localhost:5555/random](http://localhost:5555/random) to get a random available proxy.

If the download speed is particularly slow, you can modify the Dockerfile yourself, modify:

```diff
-RUN pip install -r requirements.txt
+ RUN pip install -r requirements.txt -i https://pypi.douban.com/simple
```

## Normal operation

If you do not use Docker to run, you can run it after configuring the Python and Redis environment. The steps are as follows.

### Install and configure Redis

Local installation of Redis, Docker startup of Redis, and remote Redis are all possible, as long as they can be connected and used normally.

First of all, you can need some environment variables, and the proxy pool will read these values ​​through the environment variables.

There are two ways to set Redis environment variables. One is to set host, port, and password separately, and the other is to set the connection string. The setting methods are as follows:

Set host, port, and password. If password is empty, you can set it to an empty string. Examples are as follows:

```shell script
export REDIS_HOST='localhost'
export REDIS_PORT=6379
export REDIS_PASSWORD=''
export REDIS_DB=0
```

Or just set the connection string:

```shell script
export REDIS_CONNECTION_STRING='redis://[password]@host:port/db'
```

If there is no password, set it to:

```shell script
export REDIS_CONNECTION_STRING='redis://@host:port/db'
```

The format of the connection string here needs to conform to the format of `redis://[password]@host:port/db`, be careful not to omit `@`.

Choose one of the above two settings.

### Install dependencies

[Conda](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#creating-an-environment-with-commands) is strongly recommended here
Or [virtualenv](https://virtualenv.pypa.io/en/latest/user_guide.html) to create a virtual environment, the Python version is not lower than 3.6.

Then pip installs the dependencies:

```shell script
pip3 install -r requirements.txt
```

### Run agent pool

There are two ways to run the agent pool, one is to run all Tester, Getter, and Server, and the other is to run separately on demand.

Generally you can choose to run all, the command is as follows:

```shell script
python3 run.py
```

After running, Tester, Getter, and Server will be started, then visit [http://localhost:5555/random](http://localhost:5555/random) to get a random available proxy.

Or if you figure out the structure of the proxy pool, you can run them separately as needed, the command is as follows:

```shell script
python3 run.py --processor getter
python3 run.py --processor tester
python3 run.py --processor server
```

Here the processor can specify whether to run Tester, Getter or Server.

## Use

After successful operation, you can get a random available proxy through [http://localhost:5555/random](http://localhost:5555/random).

It can be implemented by program docking. The following example shows the process of obtaining a proxy and crawling web pages:

```python
import requests

proxypool_url ='http://127.0.0.1:5555/random'
target_url ='http://httpbin.org/get'

def get_random_proxy():
    """
    get random proxy from proxypool
    :return: proxy
    """
    return requests.get(proxypool_url).text.strip()

def crawl(url, proxy):
    """
    use proxy to crawl page
    :param url: page url
    :param proxy: proxy, such as 8.8.8.8:8888
    :return: html
    """
    proxies = {'http':'http://' + proxy}
    return requests.get(url, proxies=proxies).text


def main():
    """
    main method, entry point
    :return: none
    """
    proxy = get_random_proxy()
    print('get random proxy', proxy)
    html = crawl(target_url, proxy)
    print(html)

if __name__ =='__main__':
    main()
```

The results are as follows:

```
get random proxy 116.196.115.209:8080
{
  "args": {},
  "headers": {
    "Accept": "*/*",
    "Accept-Encoding": "gzip, deflate",
    "Host": "httpbin.org",
    "User-Agent": "python-requests/2.22.0",
    "X-Amzn-Trace-Id": "Root=1-5e4d7140-662d9053c0a2e513c7278364"
  },
  "origin": "116.1
  96.115.209",
  "url": "https://httpbin.org/get"
}
```

You can see that the proxy is successfully obtained, and httpbin.org is requested to verify the availability of the proxy.

## Configurable items

The proxy pool can configure some parameters by setting environment variables.

### Switch

* ENABLE_TESTER: Allow Tester to start, default true
* ENABLE_GETTER: Allow Getter to start, default true
* ENABLE_SERVER: Run Server to start, default true

### surroundings

* APP_ENV: operating environment, you can set dev, test, prod, namely development, test, production environment, the default is dev
* APP_DEBUG: debug mode, you can set true or false, the default is true

### Redis connection

* REDIS_HOST: Redis Host
* REDIS_PORT: Redis port
* REDIS_PASSWORD: Redis password
* REDIS_DB: Redis database index, such as 0, 1
* REDIS_CONNECTION_STRING: Redis connection string
* REDIS_KEY: The name of the dictionary used by the Redis storage agent

### Processor

* CYCLE_TESTER: Tester running cycle, that is, how often to run a test, the default is 20 seconds
* CYCLE_GETTER: Getter run cycle, that is, how often to run the proxy acquisition, the default is 100 seconds
* TEST_URL: Test URL, default Baidu
* TEST_TIMEOUT: Test timeout time, the default is 10 seconds
* TEST_BATCH: the number of batch tests, the default is 20 agents
* TEST_VALID_STATUS: Is the test valid?
* API_HOST: The proxy server runs Host, default 0.0.0.0
* API_PORT: proxy server running port, default 5555
* API_THREADED: Whether the proxy server uses multiple threads, the default is true

### Log

* LOG_DIR: log relative path
* LOG_RUNTIME_FILE: Run log file name
* LOG_ERROR_FILE: Error log file name

The above content can be configured using environment variables, that is, you can set the corresponding environment variable values ​​before running, such as changing the test address and Redis key name:

```shell script
export TEST_URL=http://weibo.cn
export REDIS_KEY=proxies:weibo
```

You can build an agent pool dedicated to Weibo, and all effective agents can crawl Weibo.

If you use Docker-Compose to start the proxy pool, you need to specify environment variables in the docker-compose.yml file, such as:

```yaml
version: '3'
services:
  redis:
    image: redis:alpine
    container_name: redis
    command: redis-server
    ports:
      -"6379:6379"
    restart: always
  proxypool:
    build:.
    image:'germey/proxypool'
    container_name: proxypool
    ports:
      -"5555:5555"
    restart: always
    environment:
      REDIS_HOST: redis
      TEST_URL: http://weibo.cn
      REDIS_KEY: proxies:weibo
```

## Extended proxy crawler

The proxy crawlers are all placed under the proxypool/crawlers folder, and there are currently a limited number of proxy crawlers.

If you extend a crawler, you only need to create a new Python file in the crawlers folder to declare a Class.

The writing specifications are as follows:

```python
from pyquery import PyQuery as pq
from proxypool.schemas.proxy import Proxy
from proxypool.crawlers.base import BaseCrawler

BASE_URL ='http://www.664ip.cn/{page}.html'
MAX_PAGE = 5

class Daili66Crawler(BaseCrawler):
    """
    daili66 crawler, http://www.66ip.cn/1.html
    """
    urls = [BASE_URL.format(page=page) for page in range(1, MAX_PAGE + 1)]
    
    def parse(self, html):
        """
        parse html file to get proxies
        :return:
        """
        doc = pq(html)
        trs = doc('.containerbox table tr:gt(0)').items()
        for tr in trs:
            host = tr.find('td:nth-child(1)').text()
            port = int(tr.find('td:nth-child(2)').text())
            yield Proxy(host=host, port=port)
```

Here you only need to define a Crawler to inherit from BaseCrawler, and then define the urls variable and the parse method.

* The urls variable is the list of crawled proxy website URLs, which can be defined by programs or written as fixed content.
* The parse method receives a parameter, namely html, the html of the proxy URL. In the parse method, you only need to write the html parsing, parse out the host and port, and construct the Proxy object to yield and return.

Web crawling does not need to be implemented. BaseCrawler already has a default implementation. If you need to change the crawling method, just rewrite the crawl method.

You are welcome to send more Pull Requests and contribute to the Crawler to make its proxy source more abundant and powerful.

## Deployment

This project provides a Kubernetes deployment script. To deploy to Kubernetes, execute the following command:

```shell script
cat deployment.yml | sed's/\${TAG}/latest/g' | kubectl apply -f-
```

## To be developed

-[] Front-end page management
-[] Statistical analysis of usage

If you are interested in developing together, you can leave a message in Issue, thank you very much!

## LICENSE

MIT
