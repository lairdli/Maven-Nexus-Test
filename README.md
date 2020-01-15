## Maven-Nexus-Test

本项目用于 Maven-Nexus 私服搭建测试

前提是在Nexus配置好相关仓储

### 配置

1. build.gradle 项目根目录下添加私服地址

```groovy
allprojects {
    repositories {
        maven { url "http://192.168.3.172:8081/repository/maven-public-android/" }
        google()
        jcenter()
        
    }
}
```

2. build.gradle lib模块下添加上传脚本

```
ext.GROUP="com.avit.testlibrary"
ext.VERSION_NAME="1.1.5"
ext.POM_ARTIFACT_ID="testlibrary"

//引用gradle_maven_push.gradle
apply from: "${project.rootDir}/gradle_maven_push.gradle"

```

3. gradle_maven_push.gradle 脚本中配置maven上传
具体地址、账号以实际为准
```groovy
apply plugin: 'maven'

def MAVEN_URL = "http://192.168.3.172:8081/repository/maven-public-android/"
//release 地址
def RELEASE_REPOSITORY_URL = "http://192.168.3.172:8081/repository/maven-releases-android/"
//snapshots 地址
def SNAPSHOT_REPOSITORY_URL = "http://192.168.3.172:8081/repository/maven-snapshots-android/"
//判断 VERSION_NAME 如果以`-SHAPSHOT`结尾,则上传地址为 snapshot 地址
def REPOSITORY_URL = VERSION_NAME.endsWith("-SNAPSHOT") ? SNAPSHOT_REPOSITORY_URL : RELEASE_REPOSITORY_URL;
def NEXUS_USERNAME = "admin"
def NEXUS_PASSWORD = "admin123"
logger.info("groupId = %s\t\nartifactId = %s\t\nversion = %s\t\nrepository = %s\t\n", GROUP, POM_ARTIFACT_ID, VERSION_NAME, REPOSITORY_URL)

afterEvaluate { project ->
    uploadArchives {
        repositories.mavenDeployer {
            pom.groupId = GROUP
            pom.artifactId = POM_ARTIFACT_ID
            pom.version = VERSION_NAME
            repository(url: REPOSITORY_URL) {
                authentication(userName: NEXUS_USERNAME, password: NEXUS_PASSWORD)
            }
        }
    }
    task androidSourcesJar(type: Jar) {
        classifier = 'sources'
        from android.sourceSets.main.java.sourceFiles
    }

    artifacts {
        archives androidSourcesJar
    }
}

```
4. aar上传 

>AS-右侧工具栏-Gradle-选择具体待上传模块Tasks-upload-uploadArchives

5. aar-远程依赖 依赖
build.gradle 待依赖模块下添加依赖
```
dependencies {
    //实际建议填写具体版本号
    implementation'com.avit.testlibrary:testlibrary:+'
}

```
