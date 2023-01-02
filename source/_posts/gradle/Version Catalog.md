
---
title: Version Catalog(中央依赖声明，即：版本目录)
toc: true
date: 2022-12-28 20:16:22
tags: 
  - gradle
  - android
categories: gradle
---


# 什么是版本目录

## 1.版本目录简介

`Central declaration of dependencies` （中央依赖声明，即：版本目录）是AGP7提供的一个新特性，用于管理项目中的依赖项列表，将依赖项表示为依赖声明，随后使用者可以直接在 `build.gradle` 构建脚本使用依赖列表中的依赖声明，而不再直接使用字符串显示依赖的方式依赖。如下所示：

```kotlin
app:build.gradle

dependencies {
		// 使用implementation(libs.kotlin.core)替代implementation "androidx.core:core-ktx:1.7.0"
	  implementation(libs.coreKtx)
}
```

如上代码代码块所示， `libs.coreKtx` 是Gradle为我们自己定义的依赖目录自动生成的 `访问器`，通过它我们可以直接访问到我们定义的真正的依赖 `"androidx.core:core-ktx:1.7.0"` 而`coreKtx` 和 `"androidx.core:core-ktx:1.7.0"` 则称之为一组 `依赖目录`

在版本目录中， `libs` 为默认推荐的分组别名（也可以自定义分组别名），分组内可以声明任意多个依赖目录，`coreKtx` 则为我们自己为具体依赖项（"androidx.core:core-ktx:1.7.0"）声明的依赖别名。


<!--more-->

## 2.版本目录的优势

- Gradle会为每个依赖目录生成一个类型安全的访问器，如：`libs.coreKtx`
- 每个依赖目录对构建项目都是可见的，确保依赖项的版本适用于每个子项目或模块
- 依赖项可以声明为单个依赖目录，还可以将多个依赖项声明为 `依赖目录组`
- 依赖目录中的依赖项，可以将 `groudId:artifactId` 与 `version` 分开，将 `version` 单独声明并在依赖项中引用

# 在工程中使用版本目录

上面提到了使用 `libs.coreKtx` 替代 `"androidx.core:core-ktx:1.7.0"`，那我们应该如何声明`libs.coreKtx` ?

gradle中提供了两种构建版本目录的方式：

- 使用settings api构建版本目录
- 使用TOML文件格式构建版本目录

**使用之前**

当前Version Catalog为预览版本，并非正式版，使用之前需要在 `settings.gradle` 中单独开启功能特性

```groovy
settings.gradle

pluginManagement {
	 ...
}

// VERSION_CATALOGS当前并不是稳定版本功能
// 所以需要预先开启功能预览 enableFeaturePreview('FEATURE')
enableFeaturePreview("VERSION_CATALOGS")

dependencyResolutionManagement {
	...
}
```

## 使用 settings api

直接在项目`settings.gradle` 文件中声明依赖目录

```groovy
settings.gradle

dependencyResolutionManagement {
    
		......

    // 编写版本目录的依赖库
    versionCatalogs {
        libs {
            // 分别声明依赖别名('coreKtx')，groupId('androidx.core')，artifactId('core-ktx')以及版本('1.7.0')
            alias('coreKtx').to('androidx.core', 'core-ktx').version('1.7.0')
            alias('appcompat').to('androidx.appcompat', 'appcompat').version('1.3.0')
            alias('material').to('com.google.android.material', 'material').version('1.4.0')
            alias('constraintlayout').to('androidx.constraintlayout', 'constraintlayout').version('2.0.4')
            alias('junit-junit').to('junit', 'junit').version('4.13.2')
            alias('junit-ext').to('androidx.test.ext', 'junit').version('1.1.3')
            alias('junit-espresso').to('androidx.test.espresso', 'espresso-core').version('3.4.0')
        
						// 针对对个相同版本号的依赖，我们可以定一个通用版本号，即将依赖与版本单独声明并引用
            version('lifecycle', '2.2.0')
            alias('lifecycleExtensions').to('androidx.lifecycle', 'lifecycle-extensions').versionRef('lifecycle')
            alias('lifecycleRuntime').to('androidx.lifecycle', 'lifecycle-runtime-ktx').versionRef('lifecycle')

						// 除了单个依赖声明，我们也可以将多个依赖项声明为一个依赖组
            bundle('appBaseLib', ['coreKtx', 'appcompat', 'material', 'constraintlayout'])

            // 声明一个插件
                     // 声明一个插件
             alias('kotlin-kapt').toPluginId('org.jetbrains.kotlin.kapt').version("1.7.0")
             alias('kotlin-parcelize').toPluginId('org.jetbrains.kotlin.plugin.parcelize').version("1.7.0")
				}
    }
}
```

随即在app `build.gradle` 中使用版本目录

```groovy
plugins {
	......

	// 使用版本目录中声明的插件
	alias libs.plugins.kotlin.kapt
  alias libs.plugins.kotlin.parcelize
}

......

dependencies {
		// 依赖单个制定的版本目录
    implementation libs.coreKtx
    implementation libs.appcompat
    implementation libs.material
    implementation libs.constraintlayout

		implementation libs.lifecycleExtensions
    implementation libs.lifecycleRuntime

    testImplementation libs.junit.junit
    androidTestImplementation libs.junit.ext
    androidTestImplementation libs.junit.espresso

		// 依赖版本目录组
		// implementation libs.bundles.appBaseLib
}
```

## 使用 TOML 文件

除了在 `settings.gradle` 文件中直接声明依赖目录，官方更推荐使用 TOML 文件来声明依赖目录

首先在项目根目录下创建 `libs.versions.toml` 文件，并编写如下依赖内容：

```toml
[versions]
kotlin = "1.7.0"
appcompat = "1.3.0"
material = "1.4.0"
constraintlayout = "2.0.4"
lifecycle = "2.2.0"

[libraries]
coreKtx = { module = "androidx.core:core-ktx", version.ref = "kotlin" }
appcompat = { module = "androidx.appcompat:appcompat", version.ref = "appcompat" }
material = { module = "com.google.android.material:material", version.ref = "material" }
constraintlayout = { module = "androidx.constraintlayout:constraintlayout", version.ref = "constraintlayout" }
lifecycleExtensions = { module = "androidx.lifecycle:lifecycle-extensions", version.ref = "lifecycle" }
lifecycleRuntime = { module = "androidx.lifecycle:lifecycle-runtime-ktx", version.ref = "lifecycle" }
junit-junit = { module = "junit:junit", version = "4.13.2" }
junit_ext = { module = "androidx.test.ext:junit", version = "1.1.3" }
junit_espresso = { module = "androidx.test.espresso:espresso-core", version = "3.4.0" }

[bundles]
appBaseLib = ["coreKtx", "appcompat", "material", "constraintlayout"]

[plugins]
kotlin-kapt = { id = "org.jetbrains.kotlin.kapt", version.ref = "kotlin" }
kotlin-parcelize = { id = "org.jetbrains.kotlin.plugin.parcelize", version.ref = "kotlin" }
```

随后在 setting.gradle 中引用该 TOML 文件

```toml
settings.gradle

dependencyResolutionManagement {
    
		......

		// 第二种方式使用版本目录
    libs {
        from(files("./libs.versions.toml"))
    }
}
```

然后在app `build.gradle` 中使用 TOML 文件中声明的依赖

```groovy
......

dependencies {

		implementation libs.bundles.appBaseLib

    implementation libs.lifecycleExtensions
    implementation libs.lifecycleRuntime

    testImplementation libs.junit.junit
    androidTestImplementation libs.junit.ext
    androidTestImplementation libs.junit.espresso
}
```

接下来，我们再来详细看看 TOML 文件的一些细节。

TOML 文件由4个主要部分组成

[versions] 用于声明可以被依赖项引用的版本

[libraries] 用于声明依赖的别名

[bundles] 用于声明依赖包（依赖组）

[plugins] 用于声明插件

## 依赖远程的 TOML 文件

我们也可以将我们本地编写好的 TOML 文件发布到 maven 上，然后通过远程依赖的方式将其依赖下来使用。编辑插件的 build.gradle 文件，并将其发布到本地maven

```groovy
build.gradle

plugins {
  ......
  id 'version-catalog'
  id 'maven-publish'
}

catalog {
    // declare the aliases, bundles and versions in this block
    versionCatalog {
        from files('../libs.versions.toml')
    }
}

publishing {
    publications {
        maven(MavenPublication) {
            groupId = 'com.alg.plugin.version'
            artifactId = 'catalog'
            version = '0.0.1'
            from components.versionCatalog
        }
    }
}
```

在 setting.gradle 中依赖远程的 TOML 文件

```groovy
settings.gradle

dependencyResolutionManagement {
    
		......

		// 第三种方式使用版本目录
    libs {
        from("com.alg.plugin.version:catalog:0.0.1")
    }
}
```

然后在app `build.gradle` 中使用 TOML 文件中声明的依赖

```groovy
......

dependencies {

		implementation libs.bundles.appBaseLib

    implementation libs.lifecycleExtensions
    implementation libs.lifecycleRuntime

    testImplementation libs.junit.junit
    androidTestImplementation libs.junit.ext
    androidTestImplementation libs.junit.espresso
}
```

# 版本依赖总结
1. 关于 Gradle 的具体版本，上述测试一开始使用的 `gradle-7.3.3-bin.zip` 版本，在声明和引用 plugin 时，一直会报错，提示如下：

```
plugin request for plugin already on the classpath must not include a version
```

本来 Plugins 的声明在 7.2 以上就可以生效的，但是 7.3.3 任有问题。

建议使用使用 7.4.2 及以上版本方可解决插件不生效的问题

查看当前 gradle 版本
```
./gradlew --version 
```

将当前版本升级到7.4.2
```
./gradlew wrapper --gradle-version=7.4.2
```

2. 声明一个有效的别名
    
    别名必须由一系列标识符组成，由破折号 ( -, 推荐)、下划线 ( _) 或点 ( .) 分隔
    

groovy将会为别名自动转换为有效的访问器，且在转换过程中，会自动将别名中的`-`,`_`和`.`字符都转换为`.`，如：`groovy-core`, `groovy_json`, `groovy-nio` 将会被转换为 `groovy.core`,  `groovy.json`, `groovy.nio` ，它们均属于同一个 `groovy` 分组。

如果不希望生成子组访问器，则直接使用大写字母区分单词，而不使用`-`,`_`和`.`字符。

3. 版本目录的统一管理
    
    TOML 文件可以声明多个，官方建议如果开始使用版本目录，则应该将所有的声明都统一在 TOML文件中，外界均通过版本目录来集成所需的依赖。而依赖需要变更时，则只需修改版目录中对应的条目即可。
    
4. 关于自动补全，当前版本的Version Calalog因自身原因暂时无法自动代码补全，但是借助IDEA的插件我们可以实现自动代码补全，配置如下：

```groovy

// 根目录的build.gradle
buildscript {
    dependencies {
        classpath files(libs.class.superclass.protectionDomain.codeSource.location)
    }
}
```

5. 关于 Version Catalog 的完整demo

Version Catalog 版本目录声明仓库

[ALG Version Manager](https://github.com/ALightGroup/VersionManager)

版本目录使用

[MetaService](https://github.com/ALightGroup/MetaService)

[MetaFrame](https://github.com/ALightGroup/MetaFrame)

> 引用
> 
> [Gradle Sharing Versions](https://docs.gradle.org/current/userguide/platforms.html)