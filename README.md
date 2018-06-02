
[Set Up Jenkins Gitlab Merge Request Builder](http://klinki.github.io/set-up-jenkins-gitlab-merge-request-builder/)

In my current work we use self hosted Gitlab CE instance and Jenkins CI. We wanted to set up some integration between Gitlab and Jenkins to would build merge requests for us.

There are quite a lot of articles on the internet on that topic, but we had one major problem - our self hosted Jenkins is on private network, while Gitlab is on different, public, network.

So using webhooks and after push notifications is not going to work in that case. It took me some time to figure out how to solve that issue, but I managed to do that using polling. Since it was not really easy to do so, I will show you few steps how to solve that problem.

1) Make sure you have required Jenkins plugins installed
```
git

gitlab
```
2) Log into Gitlab and generate application API key in User Settings -> Access Tokens section

![Setup a Gitlab access token](/images/gitlab-access-token.png)

3) Configure Jenkins gitlab plugin Jenkins -> Manage Jenkins -> Configure System (or url /configure)

![Configure Jenkins Gitlab plugin](/images/gitlab-config.png)

4) Create multibranch job with scheduled executions and Configure job to use git and do branch discovery.

![Create and configure a multibranch job](/images/jenkins-branch-sources.png)

5) Use following Jenkinsfile as an inspiration


Now when you run your job, you should see in log something about Jenkinsfile found. Met criteria for each branch containing Jenkinsfile. It should also create new jobs and fire up execution on them.

```
Checking branches...
  Checking branch new-branch
      ‘Jenkinsfile’ found
    Met criteria
 No changes detected: new-branch (still at 39e856bd99dd1849554fce0504eb82ca9d1d754d)
  Checking branch testing-branch
      ‘Jenkinsfile’ found
    Met criteria
No changes detected: testing-branch (still at 94435f0a9aff1a49a9d9c1540484abfa2a584f4c)
  Checking branch from-multibranch-pipeline
      ‘Jenkinsfile’ found
    Met criteria
No changes detected: from-multibranch-pipeline (still at ff145f99798e3a0e650d3c5040d7bf71636f525a)
  Checking branch master
      ‘Jenkinsfile’ not found
    Does not meet criteria
Processed 4 branches
[Sat Feb 10 21:25:37 GMT 2018] Finished branch indexing. Indexing took 1.5 sec
Finished: SUCCESS
```
