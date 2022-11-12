
Sentry는 Application Error Monitoring 도구다. 에러 로그를 수집하는데 특화되어 있고 각 코드의 에러들을 모아 웹에서 확인할 수 있게 해주는 플랫폼이다.


#### Sentry는 다른 플랫폼에 비해서 어느 부분이 좋을까?

###### 다양한 알림 연동 지원
-   [_Azure DevOps_](https://docs.sentry.io/workflow/integrations/global-integrations/#azure-devops)_,_ [_Bitbucket_](https://docs.sentry.io/workflow/integrations/global-integrations/#bitbucket)_,_ [_GitHub_](https://docs.sentry.io/workflow/integrations/global-integrations/#github)_,_ [_GitHub Enterprise_](https://docs.sentry.io/workflow/integrations/global-integrations/#github-enterprise)_,_ [_GitLab_](https://docs.sentry.io/workflow/integrations/global-integrations/#gitlab)_,_ [_JIRA_](https://docs.sentry.io/workflow/integrations/global-integrations/#jira)_,_ [_JIRA Server_](https://docs.sentry.io/workflow/integrations/global-integrations/#jira-server)_,_ [_PagerDuty_](https://docs.sentry.io/workflow/integrations/global-integrations/#pagerduty)_,_ [_Slack_](https://docs.sentry.io/workflow/integrations/global-integrations/#slack)
-   [_Amixr_](https://docs.sentry.io/workflow/integrations/global-integrations/#amixr)_,_ [_ClickUp_](https://docs.sentry.io/workflow/integrations/global-integrations/#clickup)_,_ [_Clubhouse_](https://docs.sentry.io/workflow/integrations/global-integrations/#clubhouse)_,_ [_Rookout_](https://docs.sentry.io/workflow/integrations/global-integrations/#rookout)_,_ [_Split_](https://docs.sentry.io/workflow/integrations/global-integrations/#split)
-   [_Amazon SQS_](https://docs.sentry.io/data-management/data-forwarding/)_,_ [_Asana_](https://docs.sentry.io/workflow/integrations/legacy-integrations/#asana)_,_ Campfire*, Flowdock, [_GitLab_](https://docs.sentry.io/workflow/integrations/global-integrations/#gitlab)_,_ [_Heroku_](https://docs.sentry.io/workflow/integrations/legacy-integrations/#heroku)_,_ [_HipChat_](https://docs.sentry.io/workflow/integrations/legacy-integrations/#hipchat)_,_ Lighthouse*, OpsGenie, PagerDuty, Phabricator, Pivotal Tracker, Pushover, Redmine, [_Splunk_](https://docs.sentry.io/workflow/integrations/legacy-integrations/#splunk)_,_ Taiga, Teamwork, Trello*, Twilio
정말 다양한 플랫폼을 지원한다. 이게 가능한 이유는 Sentry에서 외부 서비스가 REST API및 WebHooks를 사용하여 Sentry Saas 서비스와 상호작용할 수 있는 방법을 제공하기 떄문이다.

###### 오류를 파악할 수 있는 다양한 정보 제공
Device: 오류가 발생한 장비 정보 (Family(Android, iOS … etc), Model, Architecture, Memory, Capacity, Simulator, BootTime, Timezone)

App: 오류가 발생한 어플리케이션 정보 (ID, Start Time, Device, Build Type, Bundle ID, Bundle Name, Version, Build)

Browser: 오류가 발생한 브라우저 정보 (Name, Version, Headers)

Operating System: 유저가 사용하는 OS (Name, Version, Kernel Version, Rooted)

BREADCRUMBS: 유저가 오류에 도달하기 까지의 경로

EXCEPTION: 에러가 발생한 코드 라인과 에러 메시지

ing....

[sentry 공식문서](https://docs.sentry.io/platforms/javascript/guides/nextjs/)
[안정성 높은 서비스 개발하기](https://medium.com/humanscape-tech/%EC%95%88%EC%A0%95%EC%84%B1-%EB%86%92%EC%9D%80-%EC%84%9C%EB%B9%84%EC%8A%A4-%EA%B0%9C%EB%B0%9C%ED%95%98%EA%B8%B0-1-2-a9e54054c675)
