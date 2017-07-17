
## 如何将aar包上传至Maven仓库
在开发二代集成灶项目的时候，系统中的所有应用的界面上都有一个统一的自定义状态栏。为了提高开发效率，我们将公共的状态栏单独抽取出来成为一个独立Model，其他应用程序引用这个Model的arr包即可。将arr包上传至maven仓库中，可以方便开发人员的集成进项目。


#### 步骤1

* 配置本地用户名、密码，配置文件为 ~/.gradle/gradle.properties，内容如下：

```
    // 域用户名、密码
    NEXUS_USERNAME=xxxx
    NEXUS_PASSWORD=xxxx
```

#### 步骤2

* 在项目工程目录下配置以下3个文件。
* 项目工程的根目录下配置： mvn.gradle (模板代码，复制进去即可)。
   
```
 

    apply plugin: 'maven'
    apply plugin: 'signing'
    
    configurations {
        archives {
            extendsFrom configurations.default
        }
    }
    
    afterEvaluate { project ->
        uploadArchives {
            repositories {
                mavenDeployer {
                    beforeDeployment { MavenDeployment deployment -> signing.signPom(deployment) }
    
                    pom.artifactId = POM_ARTIFACT_ID
    
                    repository(url: getReleaseRepositoryUrl()) {
                        authentication(userName: getRepositoryUsername(), password: getRepositoryPassword())
                    }
                    snapshotRepository(url: getSnapshotRepositoryUrl()) {
                        authentication(userName: getRepositoryUsername(), password: getRepositoryPassword())
                    }
    
                    pom.project {
                        name POM_NAME
                        packaging POM_PACKAGING
                        description POM_DESCRIPTION
                        url POM_URL
    
                        scm {
                            url POM_SCM_URL
                            connection POM_SCM_CONNECTION
                            developerConnection POM_SCM_DEV_CONNECTION
                        }
    
                        licenses {
                            license {
                                name POM_LICENCE_NAME
                                url POM_LICENCE_URL
                                distribution POM_LICENCE_DIST
                            }
                        }
    
                        developers {
                            developer {
                                id POM_DEVELOPER_ID
                                name POM_DEVELOPER_NAME
                            }
                        }
                    }
                }
            }
        }
    
        signing {
            required { isReleaseBuild() && gradle.taskGraph.hasTask("uploadArchives") }
            sign configurations.archives
        }
    }

    
   
```

* model目录下配置 gradle.properties。根据项目实际情况修改相应的value值。


  ```
  # 发布生产版本需要注释本行
  #VERSION_SNAPSHOT=SNAPSHOT 
  # 发布版本需要修改版本号
  VERSION_MAJOR=1 
  VERSION_MINOR=0.1
  
  GROUP=com.cvte.ruixin
  POM_NAME=kitchen-toolbar
  POM_ARTIFACT_ID=kitchen-toolbar
  POM_PACKAGING=aar
  POM_DESCRIPTION=CVTE ruixin's second-generation integrated stove's common toolbar.
  POM_URL=https://gitlab.gz.cvte.cn/Mid-SmartControlPanel/kitchen_toolbar
  POM_SCM_URL=https://gitlab.gz.cvte.cn/Mid-SmartControlPanel/kitchen_toolbar
  POM_SCM_CONNECTION=git@gitlab.gz.cvte.cn:Mid-SmartControlPanel/kitchen_toolbar.git
  POM_SCM_DEV_CONNECTION=git@gitlab.gz.cvte.cn:Mid-SmartControlPanel/kitchen_toolbar.git
  POM_LICENCE_NAME=The Apache Software License, Version 2.0
  POM_LICENCE_URL=http://www.apache.org/licenses/LICENSE-2.0.txt
  POM_LICENCE_DIST=repo
  POM_DEVELOPER_ID=cvte
  POM_DEVELOPER_NAME=CVTE
  
  ```


* model目录下配置 build.gradle。参考如下：

```
apply plugin: 'com.android.library'
apply from: '../mvn.gradle'

group = GROUP
version = obtainVersionName()
project.ext.set("archivesBaseName", POM_ARTIFACT_ID);


def obtainVersionName() {
    if (hasProperty('VERSION_SNAPSHOT')) {
        return "$VERSION_MAJOR.$VERSION_MINOR-SNAPSHOT"
    }
    return "$VERSION_MAJOR.$VERSION_MINOR"
}

def versionCode = (int) (Calendar.instance.timeInMillis / 1000);



android {
    compileSdkVersion 25
    buildToolsVersion "25.0.2"

    defaultConfig {
        minSdkVersion 16
        targetSdkVersion 25
        //versionCode 1
        //versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }

    defaultConfig.versionName = obtainVersionName()
    defaultConfig.versionCode = versionCode

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}


dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:25.3.1'
    // 屏幕比例适配
    compile 'com.cvte.ruixin:percent-support:1.0.0'
    testCompile 'junit:junit:4.12'
}
```

#### 步骤3 
 * 执行Gradle uploadArchieves 任务
 

```
 在你的model项目测试OK后，在IED中找到 Gradle projects列表。 
 在此列表中找到对应的model，点击Tasks --> upload --> uploadArchives。如下图所示： 
 运行完任务即可。
```

 ![gradle_task](image/gradle_task.png)
