---
title: "[CICD]Tekton in k8s Part 1"
key: 20210503
layout: article
tags: cicd tekton k8s trigger eventlist triggertemplate triggerbinding trigger
---

### 前言

前面講了一些需要注意的東西，這篇就把所有的設定都實例化來好好的講清楚。

<!--more-->

### EventListener

前面提到過這邊主要是負責事件的監聽，所有的監聽入口皆在這邊做設定。

``` yml
apiVersion: triggers.tekton.dev/v1alpha1
kind: EventListener
metadata:
  name: webhook
spec:
  serviceAccountName: pipeline
  triggers:
    - name: git-webhook-PR
      bindings:
        - ref: git-webhook-triggerbinding
      template:
        ref: git-api-trigger-template
      interceptors:
        - github:
            eventTypes: 
              - pull_request
        - cel:
            filter: "body.pull_request.merged in [true]"
            overlays:
              - key: "truncated_sha"
                expression: "truncate(body.pull_request.merge_commit_sha, 8)"
```

以上範例的相關設定在官方的catalog github裡挖的到，也有`gitlab`和`bitbucket`的範例。

這邊主要是建立`webhook`後的接收分析，經由`interceptors`去做`parse`接收`response`的格式化，主要先使用`google cel-spec`(全名`Common Expression Language`)做後續設定`key`以及正規化我要擷取的部份(這邊是把`merge commit`的`sha`擷取後只取前8位並指給`truncated_sha`這個`key`)。

接下來是把解析後的資料塞到`git-webhook-triggerbinding`設定的參數以及呼叫`git-api-trigger-template`執行接下來的動作


### TriggerBinding

先講這塊是因為這邊比較單純，可以結合上面的接收解析資料來代入範例

``` yml
---
apiVersion: triggers.tekton.dev/v1alpha1
kind: TriggerBinding
metadata:
  name: git-webhook-triggerbinding
spec:
  params:
    - name: gitrevision
      value: $(body.after)
    - name: gitrepositoryurl
      value: $(body.repository.clone_url)
    - name: revision
      value: $(body.pull_request.head.sha)
    - name: repo_owner
      value: $(body.pull_request.base.repo.owner.username)
    - name: repo_name
      value: $(body.pull_request.base.repo.name)
    - name: commit_sha
      value: $(body.pull_request.merge_commit_sha)
    - name: commit_short_sha
      value: $(expression.truncated_sha)
    - name: commit_branch
      value: $(body.pull_request.base.repo.default_branch)
    - name: commit_author
      value: $(body.pull_request.user.username)
    - name: commit_msg
      value: $(body.pull_request.body)
    - name: commit_author_email
      value: $(body.pull_request.user.email)
    - name: build_number
      value: $(body.number)
```

以上範例相關的格式也可以在官網裡翻到，這邊稍微講一下就好，主要解析的格式是上面提過的`response`內容，基本上`git`回的應該都是`json`格式，所以範例裡`body`指的就是整個`json`的本身，而這邊因為是擷取`pull request`為主，所以`$(body.pull_request.head.sha)`指的就是`response`裡的值(如下所示)

``` json
{
    "pull_request": { 
        "head": { 
            "sha": ""
        }
    }
}
```

這邊只要搞懂怎麼看就容易理解了，至於`$(expression.truncated_sha)`怎麼來的呢？就是上面`EventListener`裡額外擷取出`merge_commit_sha`前8碼給`truncated_sha`的部份。所以這一整個`TriggerBinding`就是解析整個`response`回來的`json`裡的值且依照你的設定指給你想要的參數。


### TriggerTemplate

接下來這邊就是整個`Trigger`裡最後的部份，也是牽扯最多設定的地方。

``` yml
apiVersion: triggers.tekton.dev/v1alpha1
kind: TriggerTemplate
metadata:
  name: git-api-trigger-template
spec:
  params:
    - name: gitrevision
      description: The git revision
      # default: staging
      default: master
    - name: gitrepositoryurl
      description: The git repository url
    - name: repo_owner
      description: the repository owner
    - name: repo_name
      description: the repository owner name
    - name: commit_sha
      description: git push commit sha tag
    - name: commit_short_sha
      description: git push commit sha short tag
    - name: commit_branch
      description: git push commit branch
    - name: commit_author
      description: git push commit author
    - name: commit_author_email
      description: git push commit author email
    - name: commit_msg
      description: git push commit message
    - name: build_number
      description: build number
  resourcetemplates:
    - apiVersion: tekton.dev/v1beta1
      kind: PipelineRun
      metadata:
        generateName: api-golang-stack-
        labels:
          tekton.dev/pipeline: golang-stack-pipeline
      spec:
        serviceAccountName: pipeline
        pipelineRef:
          name: golang-stack-pipeline-uat
        params:
          - name: git-repo-url
            value: '$(tt.params.gitrepositoryurl)'
          - name: git-repo-revision
            value: '$(tt.params.gitrevision)'
          - name: image-name
            # For OpenShift builds, this should be patched to point to a suitable image stream for the project
            value: your.url.path/your-project-$(tt.params.commit_branch)
          - name: secret
            value: ses-secret
          - name: mail_from
            value: 'user@mail'
          - name: mail_recipients
            value: "user@mail"
          - name: mail_repo_owner
            value: '$(tt.params.repo_owner)'
          - name: mail_repo_name
            value: '$(tt.params.repo_name)'
          - name: mail_commit_sha
            value: '$(tt.params.commit_short_sha)'
          - name: mail_commit_branch
            value: '$(tt.params.commit_branch)'
          - name: mail_commit_author
            value: '$(tt.params.commit_author)'
          - name: mail_commit_author_email
            value: '$(tt.params.commit_author_email)'
          - name: mail_commit_msg
            value: '$(tt.params.commit_msg)'
          - name: mail_build_number
            value: '$(tt.params.build_number)'
          - name: mail_build_link
            value: '$(tt.params.build_link)'
          - name: buildImageUrl
            value: your.url.path/your-project-$(tt.params.commit_branch):$(tt.params.commit_short_sha)'
        workspaces:
          - name: source
            persistentVolumeClaim:
              claimName: tekton-api-sources

```

這邊說穿了其實就只是個把`pipeline`運行之前需要的資料一一的寫入後並把需要被執行的`pipeline`指定好就可以了。

看起來好像很簡單，我自己寫起來有點像是`C++`的感覺，要先在上面的`spec.params`裡宣告好需要的參數，接著在`spec.resourcetemplates.spec.params`裡傳入對應的值，而這些值會經由`TriggerTemplate`傳入`spec.resourcetemplates.spec.pipelineRef.name`指定的`pipeline`



### 總結

這篇就到這邊，之後再找時間來講講比較細節的東西，希望以上的講解可以讓一些剛接觸的人比較好理解裡面的運作情形，如果有什麼錯誤歡迎討論。



### 參考資料

[google cel-spec](https://github.com/google/cel-spec)

[tekton catalog](https://github.com/tektoncd/catalog)
