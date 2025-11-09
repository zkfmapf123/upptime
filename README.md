# Uptime

## Status

- [x] Docker
- [x] AWS
- [x] Atlassian
- [x] Slack
- [x] Github

## Slack Alarm Actions

```sh
name: Slack Notification
on:
  workflow_dispatch:
  # schedule:
  #   - cron: "*/5 * * * *"  # 5분마다 실행 (원하면 주석처리)

jobs:
  check-and-notify:
    name: Check sites and notify Slack
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Check Docker Status
        id: docker
        continue-on-error: true
        run: |
          STATUS_CODE=$(curl -s -o /dev/null -w "%{http_code}" --max-time 10 https://www.dockerstatus.com || echo "000")
          RESPONSE_TIME=$(curl -s -o /dev/null -w "%{time_total}" --max-time 10 https://www.dockerstatus.com || echo "0")
          echo "status_code=$STATUS_CODE" >> $GITHUB_OUTPUT
          echo "response_time=$RESPONSE_TIME" >> $GITHUB_OUTPUT
          
      - name: Check AWS Status
        id: aws
        continue-on-error: true
        run: |
          STATUS_CODE=$(curl -s -o /dev/null -w "%{http_code}" --max-time 10 https://status.aws.amazon.com || echo "000")
          RESPONSE_TIME=$(curl -s -o /dev/null -w "%{time_total}" --max-time 10 https://status.aws.amazon.com || echo "0")
          echo "status_code=$STATUS_CODE" >> $GITHUB_OUTPUT
          echo "response_time=$RESPONSE_TIME" >> $GITHUB_OUTPUT

      - name: Check Atlassian Status
        id: atlassian
        continue-on-error: true
        run: |
          STATUS_CODE=$(curl -s -o /dev/null -w "%{http_code}" --max-time 10 https://status.atlassian.com || echo "000")
          RESPONSE_TIME=$(curl -s -o /dev/null -w "%{time_total}" --max-time 10 https://status.atlassian.com || echo "0")
          echo "status_code=$STATUS_CODE" >> $GITHUB_OUTPUT
          echo "response_time=$RESPONSE_TIME" >> $GITHUB_OUTPUT

      - name: Check Slack Status
        id: slack
        continue-on-error: true
        run: |
          STATUS_CODE=$(curl -s -o /dev/null -w "%{http_code}" --max-time 10 https://status.slack.com || echo "000")
          RESPONSE_TIME=$(curl -s -o /dev/null -w "%{time_total}" --max-time 10 https://status.slack.com || echo "0")
          echo "status_code=$STATUS_CODE" >> $GITHUB_OUTPUT
          echo "response_time=$RESPONSE_TIME" >> $GITHUB_OUTPUT

      - name: Check GitHub Status
        id: github
        continue-on-error: true
        run: |
          STATUS_CODE=$(curl -s -o /dev/null -w "%{http_code}" --max-time 10 https://www.githubstatus.com || echo "000")
          RESPONSE_TIME=$(curl -s -o /dev/null -w "%{time_total}" --max-time 10 https://www.githubstatus.com || echo "0")
          echo "status_code=$STATUS_CODE" >> $GITHUB_OUTPUT
          echo "response_time=$RESPONSE_TIME" >> $GITHUB_OUTPUT

      - name: Send Slack Notification
        if: always()
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
          SLACK_CHANNEL: ${{ secrets.SLACK_CHANNEL }}
        run: |
          # Docker 상태 체크
          DOCKER_STATUS="${{ steps.docker.outputs.status_code }}"
          DOCKER_TIME="${{ steps.docker.outputs.response_time }}"
          if [ "$DOCKER_STATUS" -ge 200 ] && [ "$DOCKER_STATUS" -lt 400 ]; then
            DOCKER_EMOJI="🟢"
            DOCKER_TEXT="UP"
          else
            DOCKER_EMOJI="🔴"
            DOCKER_TEXT="DOWN"
          fi

          # AWS 상태 체크
          AWS_STATUS="${{ steps.aws.outputs.status_code }}"
          AWS_TIME="${{ steps.aws.outputs.response_time }}"
          if [ "$AWS_STATUS" -ge 200 ] && [ "$AWS_STATUS" -lt 400 ]; then
            AWS_EMOJI="🟢"
            AWS_TEXT="UP"
          else
            AWS_EMOJI="🔴"
            AWS_TEXT="DOWN"
          fi

          # Atlassian 상태 체크
          ATLASSIAN_STATUS="${{ steps.atlassian.outputs.status_code }}"
          ATLASSIAN_TIME="${{ steps.atlassian.outputs.response_time }}"
          if [ "$ATLASSIAN_STATUS" -ge 200 ] && [ "$ATLASSIAN_STATUS" -lt 400 ]; then
            ATLASSIAN_EMOJI="🟢"
            ATLASSIAN_TEXT="UP"
          else
            ATLASSIAN_EMOJI="🔴"
            ATLASSIAN_TEXT="DOWN"
          fi

          # Slack 상태 체크
          SLACK_STATUS="${{ steps.slack.outputs.status_code }}"
          SLACK_TIME="${{ steps.slack.outputs.response_time }}"
          if [ "$SLACK_STATUS" -ge 200 ] && [ "$SLACK_STATUS" -lt 400 ]; then
            SLACK_EMOJI="🟢"
            SLACK_TEXT="UP"
          else
            SLACK_EMOJI="🔴"
            SLACK_TEXT="DOWN"
          fi

          # GitHub 상태 체크
          GITHUB_STATUS="${{ steps.github.outputs.status_code }}"
          GITHUB_TIME="${{ steps.github.outputs.response_time }}"
          if [ "$GITHUB_STATUS" -ge 200 ] && [ "$GITHUB_STATUS" -lt 400 ]; then
            GITHUB_EMOJI="🟢"
            GITHUB_TEXT="UP"
          else
            GITHUB_EMOJI="🔴"
            GITHUB_TEXT="DOWN"
          fi

          # Slack 메시지 생성
          TIMESTAMP=$(date -u +"%Y-%m-%d %H:%M:%S UTC")
          
          PAYLOAD=$(cat <<EOF
          {
            "channel": "${SLACK_CHANNEL}",
            "username": "Upptime Monitor",
            "icon_emoji": ":robot_face:",
            "attachments": [
              {
                "color": "#36a64f",
                "title": "🔍 SaaS Status Check Report",
                "text": "Health check completed at ${TIMESTAMP}",
                "fields": [
                  {
                    "title": "${DOCKER_EMOJI} Docker Status",
                    "value": "Status: ${DOCKER_TEXT} (${DOCKER_STATUS})\nResponse: ${DOCKER_TIME}s",
                    "short": true
                  },
                  {
                    "title": "${AWS_EMOJI} AWS Status",
                    "value": "Status: ${AWS_TEXT} (${AWS_STATUS})\nResponse: ${AWS_TIME}s",
                    "short": true
                  },
                  {
                    "title": "${ATLASSIAN_EMOJI} Atlassian Status",
                    "value": "Status: ${ATLASSIAN_TEXT} (${ATLASSIAN_STATUS})\nResponse: ${ATLASSIAN_TIME}s",
                    "short": true
                  },
                  {
                    "title": "${SLACK_EMOJI} Slack Status",
                    "value": "Status: ${SLACK_TEXT} (${SLACK_STATUS})\nResponse: ${SLACK_TIME}s",
                    "short": true
                  },
                  {
                    "title": "${GITHUB_EMOJI} GitHub Status",
                    "value": "Status: ${GITHUB_TEXT} (${GITHUB_STATUS})\nResponse: ${GITHUB_TIME}s",
                    "short": true
                  }
                ],
                "footer": "Upptime Monitor",
                "footer_icon": "https://raw.githubusercontent.com/upptime/upptime.js.org/master/static/img/icon.svg"
              }
            ]
          }
          EOF
          )

          # Slack으로 전송
          curl -X POST \
            -H 'Content-type: application/json' \
            --data "${PAYLOAD}" \
            "${SLACK_WEBHOOK_URL}"
```

