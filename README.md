# 服务器运行状态监控

显示您的监控网站的每日历史记录状态，并在您的网站状态发生变化时获取 Slack/Telegram/Discord 通知。使用 Cloudflare Workers、CRON 简单和 KV 存储。

## 准备工作

- 一个Cloudflare账户
- Cloudflare Workers ID
- Cloudflare Workers API

## 开始

### 一、Fork本仓库，并修改 README.md 中第20行仓库地址为Fork后你的仓库地址


### 二、部署在Cloudflare上

[![Deploy to Cloudflare Workers](https://camo.githubusercontent.com/1f3d0b4d44a2c3f12c78bd02bae907169430e04d728006db9f97a4befa64c886/68747470733a2f2f6465706c6f792e776f726b6572732e636c6f7564666c6172652e636f6d2f627574746f6e3f706169643d74727565)](https://deploy.workers.cloudflare.com/?url=https://github.com/fangaso/cf-workers-status-page)

1. 单击按钮并按照说明进行操作
2. 输入相关信息
2. Navigate to your new **GitHub repository &gt; Settings &gt; Secrets** and add the following secrets:

   ```yaml
   - Name: CF_API_TOKEN (should be added automatically)

   - Name: CF_ACCOUNT_ID (should be added automatically)

   - Name: SECRET_SLACK_WEBHOOK_URL (optional)
   - Value: your-slack-webhook-url

   - Name: SECRET_DISCORD_WEBHOOK_URL (optional)
   - Value: your-discord-webhook-url
   ```

3. Navigate to the **Actions** settings in your repository and enable them
4. Edit [config.yaml](./config.yaml) to adjust configuration and list all of your websites/APIs you want to monitor

   ```yaml
   settings:
     title: 'Status Page'
     url: 'https://status-page.eidam.dev' # used for Slack & Discord messages
     logo: logo-192x192.png # image in ./public/ folder
     daysInHistogram: 90 # number of days you want to display in histogram
     collectResponseTimes: false # experimental feature, enable only for <5 monitors or on paid plans

     # configurable texts across the status page
     allmonitorsOperational: 'All Systems Operational'
     notAllmonitorsOperational: 'Not All Systems Operational'
     monitorLabelOperational: 'Operational'
     monitorLabelNotOperational: 'Not Operational'
     monitorLabelNoData: 'No data'
     dayInHistogramNoData: 'No data'
     dayInHistogramOperational: 'All good'
     dayInHistogramNotOperational: 'Some checks failed'

   # list of monitors
   monitors:
     - id: workers-cloudflare-com # unique identifier
       name: workers.cloudflare.com
       description: 'You write code. They handle the rest.' # default=empty
       url: 'https://workers.cloudflare.com/' # URL to fetch
       method: GET # default=GET
       expectStatus: 200 # operational status, default=200
       followRedirect: false # should fetch follow redirects, default=false
       linkable: false # should the titles be links to the service, default=true
   ```

5. Push to `main` branch to trigger the deployment
6. 🎉
7. _\(optional\)_ Go to [Cloudflare Workers settings](https://dash.cloudflare.com/?to=/workers) and assign custom domain/route
   - e.g. `status-page.eidam.dev/*` _\(make sure you include `/*` as the Worker also serve static files\)_
8. _\(optional\)_ Edit [wrangler.toml](./wrangler.toml) to adjust Worker settings or CRON Trigger schedule, especially if you are on [Workers Free plan](#workers-kv-free-tier)

### Telegram notifications

To enable telegram notifications, you'll need to take a few additional steps.

1. [Create a new Bot](https://core.telegram.org/bots#creating-a-new-bot)
2. Set the api token you received when creating the bot as content of the `SECRET_TELEGRAM_API_TOKEN` secret in your github repository.
3. Send a message to the bot from the telegram account which should receive the alerts (Something more than `/start`)
4. Get the chat id with `curl https://api.telegram.org/bot<YOUR TELEGRAM API TOKEN>/getUpdates | jq '.result[0] .message .chat .id'`
5. Set the retrieved chat id in the `SECRET_TELEGRAM_CHAT_ID` secret variable
6. Redeploy the status page using the github action
