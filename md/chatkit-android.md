# ChatKit 使用指南 &middot; Android

[ChatKit](https://github.com/leancloud/LeanCloudChatKit-Android) 是由 LeanCloud 官方推出的、基于 LeanCloud 实时通信 SDK「LeanMessage」开发并封装了简单 UI 的聊天套件。它可以帮助开发者快速掌握 LeanMessage 的技术细节，轻松扩展和实现常用的聊天功能。

ChatKit 是一个免费且开源的项目组件，提供完全自由的授权协议，开发者可以对其进行任意的自定义和二次封装。ChatKit 的底层依然基于 LeanCloud 为各平台推出的 SDK，其最大特点是把聊天常用的一些功能配合 UI 一起提供给开发者。

## 获取项目

```bash
git clone git@github.com:leancloud/LeanCloudChatKit-Android.git
```

## 运行 Demo

获取源代码之后，用 [Android Studio](http://developer.android.com/tools/studio/index.html) 打开项目，左侧的 Project 视图显示为：

![project_structure](http://ac-lhzo7z96.clouddn.com/1459410595962)

「ChatKit-Android」Project 包含两个模块：

- **leancloudimkit**<br/>
  是一个封装了 LeanCloud 实时通讯的 UI lib，其目的是让开发者更快速地接入 LeanCloud 实时通讯的功能。
- **imkitapplication**<br/>
  为 Demo 项目，它是一个简单的示例项目，用来指导开发者如何使用 leancloudimkit。

然后，请确保电脑上已经连接了一台真机或者虚拟机用作调试。

点击 **Debug** 或者 **Run**，第一次启动会运行 Gradle Build。建议打开全局网络代理，否则 Gradle Build 可能会因为<u>网络原因</u>无法完成。


## 使用 ChatKit

开发者可以将 ChatKit 导入到自己的 Project 中使用。下面我们将新建一个 Project 用以导入 ChatKit。导入方式有三种：
- [通过 Gradle 导入](#Gradle_导入)
- [通过源代码导入](#源代码导入)
- 通过 Jar 包导入<span class="text-muted">（因为通过 Jar 包导入仍然要拷贝资源文件，所以这里不推荐此种方式。）</span>

新建一个 Application Project，取名为 **ChatDemo**。其根目录下的 `build.gradle` 的标准配置为：

```
buildscript {
    repositories {
        jcenter()
        // 这里是 LeanCloud 的包仓库
        maven {
            url "http://mvn.leancloud.cn/nexus/content/repositories/releases"
        }

    }
    dependencies {
			classpath 'com.android.tools.build:gradle:2.0.0'
    }
}

allprojects {
    repositories {
        jcenter()
        // 这里是 LeanCloud 的包仓库
        maven {
            url "http://mvn.leancloud.cn/nexus/content/repositories/releases"
        }
    }
}
```

### Gradle 导入

修改 `app/build.gradle` 文件，在 **dependencies** 中添加依赖：

```
dependencies {
    compile ('cn.leancloud.android:chatkit:1.0.0')
}
```

### 源代码导入

<div class="callout callout-info">如果是通过 Gradle 导入则不需要以下步骤。</div>

1. 浏览器访问 <https://github.com/leancloud/LeanCloudChatKit-Android>；
2. 执行以下命令行，将项目 clone 到本地（如 ChatKit 文件夹中，或者直接下载 zip 包自行解压缩到此文件夹下）：
  ```bash
  git clone https://github.com/leancloud/LeanCloudChatKit-Android.git`
  ```
3. 将文件夹 `ChatKit` 复制到 `ChatDemo` 根目录；
4. 修改 `ChatDemo/settings.gradle` 加入 `include ':ChatKit'`；
5. 修改 `ChatDemo/app/build.gradle`，在 **dependencies** 中添加 `compile project(":ChatKit")`。

最后只要 Sync Project，这样 ChatKit 就算是导入到项目中了。

### 自定义使用

一、实现自己的 Application

ChatDemo 中新建一个 Java Class，名字叫做 **ChatDemoApplication**，让它继承自 Application 类，代码如下：

```java
public class ChatDemoApplication extends Application {

// appId、appKey 可以在「LeanCloud  控制台 / 设置 / 应用 Key」获取
  private final String APP_ID = "********";
  private final String APP_KEY = "********";

  @Override
  public void onCreate() {
    super.onCreate();
    // 关于 CustomUserProvider 可以参看后面的文档
    LCChatKit.getInstance().setProfileProvider(CustomUserProvider.getInstance());
    LCChatKit.getInstance().init(getApplicationContext(), APP_ID, APP_KEY);
  }
}
```

二、在 `AndroidMainfest.xml` 中配置 ChatDemoApplication

```xml
<application
  ...
  android:name=".ChatDemoApplication" >
  ...
</application>
```

三、实现自己的用户体系

ChatDemo 中新建一个 Java Class，名字叫做 **CustomUserProvider**，代码如下：

```java
public class CustomUserProvider implements LCChatProfileProvider {

  private static CustomUserProvider customUserProvider;

  public synchronized static CustomUserProvider getInstance() {
    if (null == customUserProvider) {
      customUserProvider = new CustomUserProvider();
    }
    return customUserProvider;
  }

  private CustomUserProvider() {
  }

  private static List<LCChatKitUser> partUsers = new ArrayList<LCChatKitUser>();

  // 此数据均为模拟数据，仅供参考
  static {
    partUsers.add(new LCChatKitUser("Tom", "Tom", "http://www.avatarsdb.com/avatars/tom_and_jerry2.jpg"));
    partUsers.add(new LCChatKitUser("Jerry", "Jerry", "http://www.avatarsdb.com/avatars/jerry.jpg"));
    partUsers.add(new LCChatKitUser("Harry", "Harry", "http://www.avatarsdb.com/avatars/young_harry.jpg"));
    partUsers.add(new LCChatKitUser("William", "William", "http://www.avatarsdb.com/avatars/william_shakespeare.jpg"));
    partUsers.add(new LCChatKitUser("Bob", "Bob", "http://www.avatarsdb.com/avatars/bath_bob.jpg"));
  }

  @Override
  public void fetchProfiles(List<String> list, LCChatProfilesCallBack callBack) {
    List<LCChatKitUser> userList = new ArrayList<LCChatKitUser>();
    for (String userId : list) {
      for (LCChatKitUser user : partUsers) {
        if (user.getUserId().equals(userId)) {
          userList.add(user);
          break;
        }
      }
    }
    callBack.done(userList, null);
  }

  public List<LCChatKitUser> getAllUsers() {
    return partUsers;
  }
}
```

四、打开实时通讯，并且跳转到聊天页面。

```java
LCChatKit.getInstance().open("Tom", new AVIMClientCallback() {
      @Override
      public void done(AVIMClient avimClient, AVIMException e) {
        if (null == e) {
          finish();
          Intent intent = new Intent(MainActivity.this, LCIMConversationActivity.class);
          intent.putExtra(LCIMConstants.PEER_ID, "Jerry");
          startActivity(intent);
        } else {
          Toast.makeText(MainActivity.this, e.toString(), Toast.LENGTH_SHORT).show();
        }
      }
    });
```

这样，Tom 就可以和 Jerry 愉快地聊天了。

## 接口以及组件

ChatKit 中开发者常需要关注的业务逻辑组件（Interface）和界面组件（UI）有：

### 用户

`LCChatKitUser` 是 ChatKit 封装的参与聊天的用户，它提供了如下属性：

名称 | 描述
--- | ---
`userId` | 用户在单个应用内唯一的 ID
`avatarUrl` | 用户的头像
`name` | 用户名

使用这些默认的属性基本可以满足一个聊天应用的需求，同时开发者可以通过继承 `LCChatKitUser` 类实现更多属性。具体用法请参考 Demo 中的 `MembersAdapter.java`。

### 用户信息管理类

`LCChatProfileProvider` 接口需要用户 implements 后实现 `fetchProfiles` 函数，以使 ChatKit 在需要显示的时候展示用户相关的信息。

例如 Demo 中的 `CustomUserProvider` 这个类，它实现了 `LCChatProfileProvider` 接口，在 `fetchProfiles` 方法里加载了 Tom、Jerry 等人的信息。

### 核心类 

`LCChatKit` 是 ChatKit 的核心类，具体逻辑可以参看代码，注意以下几个主要函数：

<dl>
<dt>`public void init(Context context, String appId, String appKey)`</dt>
<dd>此函数用于初始化 ChatKit 的相关设置，此函数要在 Application 的 `onCreate` 中调用，否则可能会引起异常。</dd>
<dt>`public void setProfileProvider(LCChatProfileProvider profileProvider)`</dt>
<dd>此函数用于设置用户体系，因为 LeanCloud 实时通讯功能已经实现了完全剥离用户体系的功能，这里接入已有的用户体系会很方便。</dd>
<dt>`public void open(final String userId, final AVIMClientCallback callback)`</dt>
<dd>此函数用于开始实时通讯，open 成功后可以执行自己的逻辑，或者跳转到聊天页面。</dd>
</dl>

### 对话列表界面

对话 `AVIMConversation` 是 LeanMessage 封装的用来管理对话中的成员以及发送消息的载体，不论是群聊还是单聊都是在一个对话当中；而对话列表可以作为聊天应用默认的首页显示出来，主流的社交聊天软件，例如微信，就是把最近的对话列表作为登录后的首页。

因此，我们也提供了对话列表 `LCIMConversationListFragment` 页面供开发者使用，在 Demo 项目中的 `MainActivity` 中的 `initTabLayout` 方法中演示了如何引入对话列表页面：

```java
  private void initTabLayout() {
    String[] tabList = new String[]{"会话", "联系人"};
    final Fragment[] fragmentList = new Fragment[] {new LCIMConversationListFragment(),
            new ContactFragment()};
    // 以上这段代码为新建了一个 Fragment 数组，并且把 LCIMConversationListFragment 作为默认显示的第一个 Tab 页面

    tabLayout.setTabMode(TabLayout.MODE_FIXED);
    for (int i = 0; i < tabList.length; i++) {
      tabLayout.addTab(tabLayout.newTab().setText(tabList[i]));
    }
    ...
}
```
具体的显示效果如下：

<img class="img-bordered" src="https://dn-lhzo7z96.qbox.me/1464320206432">

### 聊天界面

聊天界面是显示频率最高的前端页面，ChatKit 通过 `LCIMConversationFragment` 和 `LCIMConversationActivity` 来实现这一界面。在 Demo 的 `ContactItemHolder` 界面包含了使用 `LCIMConversationActivity` 的实例：

```java
  public void initView() {
    nameView = (TextView)itemView.findViewById(R.id.tv_friend_name);
    avatarView = (ImageView)itemView.findViewById(R.id.img_friend_avatar);

    itemView.setOnClickListener(new View.OnClickListener() {
      @Override
      public void onClick(View v) {
        // 点击联系人，直接跳转进入聊天界面 
        Intent intent = new Intent(getContext(), LCIMConversationActivity.class);
        // 传入对方的 Id 即可
        intent.putExtra(LCIMConstants.PEER_ID, lcChatKitUser.getUserId());
        getContext().startActivity(intent);
      }
    });
  }
```
具体的显示效果如下：

<img class="img-bordered" src="https://dn-lhzo7z96.qbox.me/1461816927540">

## 常见问题

**ChatKit 组件收费么？**<br/>
ChatKit 是完全开源并且免费给开发者使用，使用聊天所产生的费用以账单为准。

**接入 ChatKit 有什么好处？**<br/>
它可以减轻应用或者新功能研发初期的调研成本，直接引入使用即可。ChatKit 从底层到 UI 提供了一整套的聊天解决方案。
