<p align="left">
  <a target="_blank" href="https://github.com/open-dingtalk/dingtalk-stream-sdk-python/actions/workflows/publish.yml">
    <img src="https://img.shields.io/github/actions/workflow/status/open-dingtalk/dingtalk-stream-sdk-python/publish.yml" />
  </a>

  <a target="_blank" href="https://pypi.org/project/dingtalk-stream/">
    <img alt="PyPI - Version" src="https://img.shields.io/pypi/v/dingtalk-stream">
  </a>
</p>

# DingTalk Stream Mode 介绍

Python SDK for DingTalk Stream Mode API, Compared with the webhook mode, it is easier to access the DingTalk chatbot

钉钉支持 Stream 模式接入事件推送、机器人收消息以及卡片回调，该 SDK 实现了 Stream 模式。相比 Webhook 模式，Stream 模式可以更简单的接入各类事件和回调。

## 快速指南

1. 安装 SDK

```Python
pip install dingtalk-stream
```

2. 开发一个 Stream 机器人

以下示例注册一个加法处理会调，可以实现一个加法机器人（给机器人发送1+1，回复2）

```Python
# !/usr/bin/env python

import argparse
import logging
from dingtalk_stream import AckMessage
import dingtalk_stream

def setup_logger():
    logger = logging.getLogger()
    handler = logging.StreamHandler()
    handler.setFormatter(
        logging.Formatter('%(asctime)s %(name)-8s %(levelname)-8s %(message)s [%(filename)s:%(lineno)d]'))
    logger.addHandler(handler)
    logger.setLevel(logging.INFO)
    return logger


def define_options():
    parser = argparse.ArgumentParser()
    parser.add_argument(
        '--client_id', dest='client_id', required=True,
        help='app_key or suite_key from https://open-dev.digntalk.com'
    )
    parser.add_argument(
        '--client_secret', dest='client_secret', required=True,
        help='app_secret or suite_secret from https://open-dev.digntalk.com'
    )
    options = parser.parse_args()
    return options


class CalcBotHandler(dingtalk_stream.ChatbotHandler):
    def __init__(self, logger: logging.Logger = None):
        super(dingtalk_stream.ChatbotHandler, self).__init__()
        if logger:
            self.logger = logger

    async def process(self, callback: dingtalk_stream.CallbackMessage):
        incoming_message = dingtalk_stream.ChatbotMessage.from_dict(callback.data)
        expression = incoming_message.text.content.strip()
        try:
            result = eval(expression)
        except Exception as e:
            result = 'Error: %s' % e
        self.logger.info('%s = %s' % (expression, result))
        response = 'Q: %s\nA: %s' % (expression, result)
        self.reply_text(response, incoming_message)

        return AckMessage.STATUS_OK, 'OK'

def main():
    logger = setup_logger()
    options = define_options()

    credential = dingtalk_stream.Credential(options.client_id, options.client_secret)
    client = dingtalk_stream.DingTalkStreamClient(credential)
    client.register_callback_handler(dingtalk_stream.chatbot.ChatbotMessage.TOPIC, CalcBotHandler(logger))
    client.start_forever()


if __name__ == '__main__':
    main()
```

## 高阶使用方法

以上示例中，采用 `client.start_forever()` 来启动一个 asyncio 的 ioloop。

有的时候，你需要在已有的 ioloop 中使用钉钉 Stream 模式，不使用 `start_forever` 方法。

此时，可以使用 `client.start()` 代替 `client.start_forever()`。注意：需要在网络异常后重新启动

```Python
try:
    await client.start()
except (asyncio.exceptions.CancelledError,
        websockets.exceptions.ConnectionClosedError) as e:
    ... # 处理网络断线异常
```

## 开发教程

在 [教程文档](https://opensource.dingtalk.com/developerpedia/docs/explore/tutorials/stream/overview) 中，你可以找到更多钉钉 Stream 模式的教程文档和示例代码。

## 特别说明

因拼写错误，从旧版本升级到 v0.13.0 时候，需要将 register_callback_hanlder 修改为 register_callback_handler

### 参考资料

* [Stream 模式说明](https://opensource.dingtalk.com/developerpedia/docs/learn/stream/overview)
* [教程文档](https://opensource.dingtalk.com/developerpedia/docs/explore/tutorials/stream/overview)
* [常见问题](https://opensource.dingtalk.com/developerpedia/docs/learn/stream/faq)
* [Stream 模式共创群](https://opensource.dingtalk.com/developerpedia/docs/explore/support/?via=moon-group)
