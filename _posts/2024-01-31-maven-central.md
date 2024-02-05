---
title: "[MSA 4] Maven Repository로 MSA 환경에서 DTO 공유하기(maven central)"
description: nexus repository로 maven central에 공통 코드 등록하여 msa환경에서 코드 공유하기
date: 2024-01-31 10:38 +0900
lastmod: 2024-01-31 10:38 +0900
categories: msa
tags:
  - maven central
  - msa
  - nexus repository
  - common dto
---

maven central은 여러가지 dependency를 저장해 놓은 패키지 저장소이다. 프로젝트에 필요한 의존성을 import 하여 사용할 수 있다.

## MSA 환경에서 Maven repository의 필요성

MSA 환경에서는 DTO(Data Transfer Object)를 공유하게 되는 경우가 발생한다. 마이크로 서비스 간의 통신이나 이벤트 발생 시 데이터를 publish, consume 하는 경우 동일한 DTO를 사용해야 한다. 또한 여러 마이크로 서비스에서 공통으로 사용되는 로직이 존재한다면 maven repository를 사용해 한번의 수정으로 필요한 마이크로 서비스들에 적용할 수 있다.

## Maven central 저장소 올리기

maven repository를 생성하기 위해 먼저 Jira 이슈를 생성해야 한다. [sonatype](https://issues.sonatype.org/secure/Dashboard.jspa) 에 접속하여 시작한다.

![Screenshot 2023-12-10 at 03 55 57](https://github.com/nowgnas/nowgnas.github.io/assets/55802893/23d77f84-03f9-4d50-b9f6-bf18156178f2)

위 이미지는 sonatype jira 페이지의 상단의 내용이다. OSSRH와 maven central을 위한 jira라고 한다.
![Screenshot 2023-12-10 at 03 57 31](https://github.com/nowgnas/nowgnas.github.io/assets/55802893/96e102eb-b82e-4940-a9b2-e5871411e4d6)

상단에 `만들기` 버튼으로 jira 이슈를 생성한다. 프로젝트는 하나로 고정되어 있어 신경쓸 필요 없다.

- 요약: jira title
- 설명: 간단한 설명
- Group Id: io.github.\<프로젝트 주소 도메인\>
- project url: github repo url을 작성했다
- scm url: 코드가 관리될 repo
- username: 사용자 이름

위 정보를 입력하여 생성해주면 된다.
몇 분 뒤 jira의 봇이 답변을 해준다. 답변을 잘 읽고 시키는 것을 따라하면 된다. 보통 이슈 번호에 맞는 repo를 생성하거나 group id를 수정하라고 한다.
![Screenshot 2023-12-10 at 04 04 42](https://github.com/nowgnas/nowgnas.github.io/assets/55802893/5e622cb0-c2ec-41bc-88a0-7e861f0b319f)

repo를 생성하고 시키는 것을 완료하면 답변을 남겨준다. 답변은 아무렇게나 해도 괜찮은것 같다.
![Screenshot 2023-12-10 at 04 05 53](https://github.com/nowgnas/nowgnas.github.io/assets/55802893/aba7e483-83c9-4d11-90e4-19d945d216b8)

시키는 것을 잘 하면 위와 같이 축하와 함께 maven central에 릴리즈 할 수 있도록 공식 문서로 안내해준다. 이제 gradle 프로젝트를 생성하고 배포하면 된다.

## Gradle 프로젝트 생성하기

![img1](https://github.com/nowgnas/nowgnas.github.io/assets/55802893/30cee9af-361b-4c99-8bfd-02bdcaf2c4f0)

intellij 에서 new project 선택 후 build system을 gradle로 설정하여 프로젝트를 생성해준다.
![img2](https://github.com/nowgnas/nowgnas.github.io/assets/55802893/bfd1b6d1-a561-467b-bfb8-c9f67be3fba4)

build.gradle로 빌드 후 publish 하게 된다. 배포하기 위해 필요한 파일을 만들어줘야 한다.

```shell
publish-maven.gradle
local.properties (add gitignore)
publish.gradle
```

[전체프로젝트](https://github.com/lotteon2/BB-COMMON-REPOSITORY)

### publish.gradle

```groovy
ext {
    PUBLISH_GROUP_ID = ''
    PUBLISH_VERSION = (int)(((new Date().getTime()/1000) - 1451606400) / 10) // 버전 관리 귀찮아서 날짜로 대체
    PUBLISH_ARTIFACT_ID = 'blooming-blooms-utils'
    PUBLISH_DESCRIPTION = 'common repository for blooming blooms project'
    PUBLISH_URL = 'https://github.com/lotteon2/BB-COMMON-REPOSITORY'
    PUBLISH_LICENSE_NAME = 'GNU License'
    PUBLISH_LICENSE_URL = 'https://github.com/lotteon2/BB-COMMON-REPOSITORY/blob/main/LICENSE'
    PUBLISH_DEVELOPER_ID = 'nowgnas'
    PUBLISH_DEVELOPER_NAME = 'sangwon'
    PUBLISH_DEVELOPER_EMAIL = 'dev.nowgnas@gmail.com'
    PUBLISH_SCM_CONNECTION = 'scm:git:github.com/lotteon2/BB-COMMON-REPOSITORY.git'
    PUBLISH_SCM_DEVELOPER_CONNECTION = 'scm:git:ssh://github.com:lotteon2/BB-COMMON-REPOSITORY.git'
    PUBLISH_SCM_URL = 'https://github.com/lotteon2/BB-COMMON-REPOSITORY/tree/main'
}
```

배포를 위한 정보들이다. 이 값들을 `publish-maven.gradle` 에서 사용하게 된다. 배포 버전은 배포할 때마다 매번 변경해도 되고 날짜에 따라 자동 생성되도록 해도 된다. (버전을 안바꾸고 올리면 적용이 잘 안되는 것 같아 날짜로 했습니다.)

## GPG key 발급 및 서버 등록

```shell
gpg --gen-key
```

위 명령어로 gpg key를 생성한다. 키를 생성한 후 gpg key를 관리하는 서버에 전송해야 한다.

```shell
gpg --keyserver http://keyserver.ubuntu.com --send-keys <public_key>
```

```shell
gpg --export-secret-keys -o secring.gpg
```

위 명령어로 gpg 파일을 만들어 준다.

## maven central 에 프로젝트 배포하기

maven central에 프로젝트를 배포하기 위해 프로젝트를 빌드하고, nexus repository에 업로드 후 maven central에 배포된다. 배포 과정을 알아본다.

### nexus repository에 업로드 하기

local.properties 파일을 프로젝트 root에 생성한다

```properties
signing.keyId=<gpg key 8개 뒷자리>
signing.password=<gpg key password>
signing.secretKeyRingFile=<gpg key 경로>

ossrhUsername=<username>
ossrhPassword=<password>
sonatypeStagingProfileId=<profile id>
```

![img3](https://github.com/nowgnas/nowgnas.github.io/assets/55802893/546fd823-ec02-4cf5-9793-8b414f949d79)

nexus repository에 업로드하기 위한 gradle 파일을 생성한다.

```groovy
apply plugin: 'maven-publish'
apply plugin: 'signing'
apply from: 'publish.gradle' // 위에서 생성한 publish.gradle

task sourcesJar(type: Jar) {
    archiveClassifier.set("sources")
}

task javadocJar(type: Jar) {
    archiveClassifier.set("javadoc")
}

artifacts {
    archives sourcesJar
    archives javadocJar
}


group = PUBLISH_GROUP_ID
version = PUBLISH_VERSION

ext["signing.keyId"] = ''
ext["signing.password"] = ''
ext["signing.secretKeyRingFile"] = ''
ext["ossrhUsername"] = ''
ext["ossrhPassword"] = ''
ext["sonatypeStagingProfileId"] = ''

File secretPropsFile = project.rootProject.file('local.properties')
if (secretPropsFile.exists()) {
    Properties p = new Properties()
    p.load(new FileInputStream(secretPropsFile))
    p.each { name, value ->
        ext[name] = value
    }
} else {
    ext["signing.keyId"] = System.getenv('SIGNING_KEY_ID')
    ext["signing.password"] = System.getenv('SIGNING_PASSWORD')
    ext["signing.secretKeyRingFile"] = System.getenv('SIGNING_SECRET_KEY_RING_FILE')
    ext["ossrhUsername"] = System.getenv('OSSRH_USERNAME')
    ext["ossrhPassword"] = System.getenv('OSSRH_PASSWORD')
    ext["sonatypeStagingProfileId"] = System.getenv('SONATYPE_STAGING_PROFILE_ID')
}

publishing {
    publications {
        release(MavenPublication) {
            groupId PUBLISH_GROUP_ID
            artifactId PUBLISH_ARTIFACT_ID
            version PUBLISH_VERSION

            artifact("$buildDir/libs/${project.getName()}-${version}.jar")
            artifact sourcesJar
            artifact javadocJar

            pom {
                name = PUBLISH_ARTIFACT_ID
                description = PUBLISH_DESCRIPTION
                url = PUBLISH_URL
                licenses {
                    license {
                        name = PUBLISH_LICENSE_NAME
                        url = PUBLISH_LICENSE_URL
                    }
                }                developers {
                    developer {
                        id = PUBLISH_DEVELOPER_ID
                        name = PUBLISH_DEVELOPER_NAME
                        email = PUBLISH_DEVELOPER_EMAIL
                    }
                    // Other devs...
                }
                scm {
                    connection = PUBLISH_SCM_CONNECTION
                    developerConnection = PUBLISH_SCM_DEVELOPER_CONNECTION
                    url = PUBLISH_SCM_URL
                }
                withXml {
                    def dependenciesNode = asNode().appendNode('dependencies')

                    project.configurations.implementation.allDependencies.each {
                        def dependencyNode = dependenciesNode.appendNode('dependency')
                        dependencyNode.appendNode('groupId', it.group)
                        dependencyNode.appendNode('artifactId', it.name)
                        dependencyNode.appendNode('version', it.version)
                    }
                }            }        }    }    repositories {
        maven {
            name = "sonatype"
            url = "https://s01.oss.sonatype.org/service/local/staging/deploy/maven2/"

            credentials {
                username ossrhUsername
                password ossrhPassword
            }
        }    }}

nexusPublishing {
    repositories {
        sonatype {
            nexusUrl.set(uri("https://s01.oss.sonatype.org/service/local/"))
            snapshotRepositoryUrl.set(uri("https://s01.oss.sonatype.org/content/repositories/snapshots/"))
            packageGroup = PUBLISH_GROUP_ID
            stagingProfileId = sonatypeStagingProfileId
            username = ossrhUsername
            password = ossrhPassword
        }
    }}

signing {
    sign publishing.publications
}
```

전체 코드는 위와 같다. 여러 micro service에서 사용할 코드를 빌드 후 nexus repository에 배포하면 된다.

```bash
./gradlew clean build # build

./gradlew publishReleasePublicationToSonatypeRepository # publish
```

위 명령어를 실행하게 되면 nexus repository의 staging repository로 올라가게 된다.
![img4](https://github.com/nowgnas/nowgnas.github.io/assets/55802893/5bac93a0-958b-43e6-a62c-454fc85d86db)

staging repository에서 업로드된 repo를 close 한 후 release하면 maven central에 의존성이 등록된다.
![img5](https://github.com/nowgnas/nowgnas.github.io/assets/55802893/0fe5a057-a64b-464b-bf62-f624d6b6ee5f)

release를 클릭해 의존성을 배포한다. 대략 10~20분 후 maven central에서 의존성을 확인할 수 있다. maven central에서 검색할 때는 group id나 artifact id 로 검색하면 된다.

이제 micro service에서 공통으로 사용되는 코드를 maven central에 등록해 사용할 수 있다.
