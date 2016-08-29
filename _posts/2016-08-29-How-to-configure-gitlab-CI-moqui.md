---
layout: post
title: How to configure Gitlab CI for Moqui
permalink: /github-ci-config-moqui/
author:    locnx
tags: gitlab ci moqui continous-integration
category: Moqui
---

### Background

You've developed a Moqui component, which has name _yourcomponent_, and push it into a Gitlab repo.

To enable Continuous Integration for this component, we need to:

1. Add `.gitlab-ci.yml` to the root directory of your repository
2. Configure a Runner, or select a shared one

<!--more-->

### Configuration

`.gitlab-ci.yml`

```yml
image: java:8-jdk

stages:
  - build

build:
# NOTE: using git clone instead of gradle getGit for file order, causing problems on Travis with test class run order
  before_script:
  - export GRADLE_USER_HOME=`pwd`/.gradle
  - cd ..
  - git clone https://github.com/moqui/moqui-framework.git moqui
  - cd moqui
  - git clone https://github.com/moqui/moqui-runtime.git runtime
  - cd runtime/component
  - git clone https://github.com/moqui/moqui-elasticsearch.git
  - git clone https://github.com/moqui/moqui-fop.git
  - git clone https://github.com/moqui/moqui-kie.git
  - git clone https://github.com/moqui/mantle-udm.git
  - git clone https://github.com/moqui/mantle-usl.git
  - git clone https://github.com/moqui/SimpleScreens.git
  - git clone https://github.com/moqui/PopCommerce.git
  - mv -f $CI_PROJECT_DIR .
  - cd ../..
  - chmod +x gradlew

  stage: build
  script:
  # NOTE: replace 'yourcomponent' with your real component name
  - ./gradlew load
  - ./gradlew runtime/component/yourcomponent:test  

cache:
  paths:
    - $GRADLE_USER_HOME/.gradle/caches
    - $GRADLE_USER_HOME/.gradle/wrapper
```

### View test result

Follow Gitlab guide about Gitlab CI: [http://docs.gitlab.com/ce/ci/quick_start/README.html](http://docs.gitlab.com/ce/ci/quick_start/README.html)
