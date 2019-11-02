## 1、模板文件结构
AndroidStudio中模板全部放在<font color='red'>/plugins/android/lib/templates/</font>中


- template.xml(<font color='red'>创建面板获取输入参数</font>)
- globals.xml.ftl(<font color='red'>全局变量参数</font>)
- recipe.xml.ftl(<font color='red'>主要用于生成我们实际需要的代码，资源文件等</font>)
- root(<font color='red'>存放对应源码的ftl文件和资源文件</font>)
- template_blank_activity.png(<font color='red'>模板效果预览图</font>)

### 1.1、template.xml

```xml
<?xml version="1.0"?>
<!--
    format：相当于andorid versionName
    revision：(可选)相当于android versionCode
    name：此模板显示的名称
    minApi：(可选)此模板要求的最低版本api，minSdkVersion不能低于此值
    minBuildApi：(可选)此模板要求的最低目标版本api，targetSdkVersion不能低于此值，确保可以使用最新的api
    description：此模板的描述
    -->
<template
    format="5" 
    revision="5"
    name="Empty Activity"
    minApi="9"
    minBuildApi="14"
    description="Creates a new empty activity">
    
    <!--如果项目中不存在v4库，则向项目中添加，所需v4库最低版本为8-->
    <dependency name="android-support-v4" revision="8" />

    <!--此模板类型，可选值有Application、Activity、UI Component-->
    <category value="Activity" />
    
    <!--平台，可选值有Mobile、TV、Wear、Car-->
    <formfactor value="Mobile" />

    <!--用户可定义的参数-->
    <parameter
        id="activityClass"
        name="Activity Name"
        type="string"
        constraints="class|unique|nonempty"
        suggest="${layoutToActivity(layoutName)}"
        default="MainActivity"
        help="The name of the activity class to create" 
        visibility="isInstantApp!false"
        enabled="includeInstantAppUrl">
        <option id="path1">Path1</option>
        <option id="path2">Path2</option>
        <option id="path3">Path3</option>
    </parameter>
        <!--
        id：相当于view的id，通过id获取此属性，使用:${activityClass}
        name：模板参数显示名称
        type：输入类型，可选值有stirng、int、boolean、enum、separator
        suggest：建议值
        default：默认值
        help：在创建面板下方显示的提示
        visibility：可见性可使用表达式。比如:isInstantApp!false
        enabled：是否启用。比如:includeInstantAppUrl(boolean)
        option：枚举值
        constraints：(可选)对参数添加约束
                    nonempty：不能为空
                    apilevel：api级别
                    package：该值为包名
                    class：该值为类名
                    layout：该值为布局名称
                    drawable：该值为drawable
                    string：该值为字符串资源名称
                    id：该值为有效的id名称
                    unique：唯一限制。比如布局，不能输入已经存在的布局名称
                    exists：该值必须存在。比如布局，应该输入已经存在的布局名称
        -->
    <!--模板的缩略图-->
    <thumbs>
        <thumb>template_blank_activity.png</thumb>
    </thumbs>

    <globals file="globals.xml.ftl" />
    <execute file="recipe.xml.ftl" />

</template>
```
> 使用到的函数<font color='red'>${layoutToActivity(layoutName)}</font>

作用是将Activity的名字转为规范的layout名，比如参数MainActivity就会返回activity_main。除此之外，还有许多常用的函数：

- <font color='red'>camelCaseToUnderscore(string)</font>：驼峰命名风格转下划线
- <font color='red'>escapeXmlAttribute(string)</font>：此函数用来转义字符串，例如 Android's，它可以用作 XML 属性值：Android&apos;s。特别是，它将转义'，"，< 和 ＆
- <font color='red'>escapeXmlText(string)</font>：此函数用来转义字符串，例如 A & B's 可以将其用作 XML 文本。这意味着它将转义 < 和 >，但不像 escapeXmlAttribute 它将不会转义' 和 "。在前面的示例中，它将转义字符串为 A &amp; B\s。请注意，如果您想要使用 XML 文本作为 <string> 资源的值，您应该考虑使用 escapeXmlString，因为它执行额外的所需的字符串资源转义
- <font color='red'>escapeXmlString(string)</font>：此函数用来转义字符串，例如 A & B's ，它适合作为 XML 文本插入字符串资源文件中，例如 A &amp; B\s 。除了转义 < 和 ＆ 之类的 XML 字符外，它还执行其他Android 特定的转义，例如使用反斜杠转义撇号
- <font color='red'>extractLetters(string)</font>：此函数从字符串中提取所有字母，有效删除任何标点符号和空白字符
- <font color='red'>classToResource(string)</font>：把Activity转换为资源标识符。比如MainActivity会转换为main
- <font color='red'>layoutToActivity(string)</font>：将资源标识符转换为Java字符。比如activity_main会转换为“MainActivity”
- <font color='red'>slashedPackageName(string)</font>：包名转路径。比如：com.example.foo返回com/example/foo
- <font color='red'>underscoreToCamelCase(string)</font>：下划线转驼峰



![image](http://q064lui49.bkt.clouddn.com/template_panel)
### 1.2、globals.xml.ftl

```xml
<?xml version="1.0"?>
<globals>
    <global id="hasNoActionBar" type="boolean" value="false" />
    <global id="simpleLayoutName" value="${layoutName}" />
    <#include "../common/common_globals.xml.ftl" />
</globals>
```
全局变量文件，存放的是一些全局变量。可通过id获取到该值。如：<font color='red'>${hasNoActionBar}</font>


### 1.3、recipe.xml.ftl
```xml
<?xml version="1.0"?>
<recipe>
    <#if !(hasDependency('com.android.support:appcompat-v7'))>
       <dependency mavenUrl="com.android.support:appcompat-v7:${buildApi}.+" />
    </#if>
    <copy from="root/res/drawable-hdpi"
            to="${escapeXmlAttribute(resOut)}/drawable-hdpi" />
    <merge from="root/AndroidManifest.xml.ftl"
             to="${escapeXmlAttribute(manifestOut)}/AndroidManifest.xml" />
    <instantiate from="root/res/layout/activity_login.xml.ftl"
                   to="${escapeXmlAttribute(resOut)}/layout/${layoutName}.xml" />
    <open file="${escapeXmlAttribute(srcOut)}/${activityClass}.${ktOrJavaExt}" />
</recipe>
```
##### 标签说明：
- <font color=red><copy></font>：从root中copy文件到我们的目录,比如我们的模板Activity需要使用一些图标，那么可能就需要使用copy标签将这些图标拷贝到我们的项目对应文件夹。
- <font color=red><merge></font>：合并文件，比如将string.xml和并到我们的string.xml
- <font color=red><instantiate></font>：和copy类似，但是可以看到上例试将ftl->java文件的，也就是说中间会通过一个步骤，将ftl中的变量都换成对应的值，那么完整的流程是<font color='red'>ftl->freemarker process -> java</font>。
- <font color=red><open></font>：生成后打开指定的文件，比如我们新建一个Activity，就会把这个Activity打开
- <font color=red><dependency></font>：依赖库，会自动下载指定的library

<#if >为FreeMarker条件表达式，可参考[FreeMarker在线手册](http://freemarker.foofun.cn/)

## 2、示例
现在来编写一个带有RecyclerView的Activity，练习一下基本的用法
### 2.1、template.xml的编写
这里为了演示基础用法，添加了布局名称、RecyclerView的item个数、是否显示分割线、布局类型等几个参数
![image](http://q064lui49.bkt.clouddn.com/template_sample.png)
```xml
<?xml version="1.0"?>
<template
        name="RecyclerView Activity"
        description="创建一个列表Activity">

    <category value="Activity"/>
    <formfactor value="Mobile"/>

    <parameter
            id="activityClass"
            name="Activity Name"
            type="string"
            constraints="class|unique|nonempty"
            suggest="${layoutToActivity(layoutName)}"
            default="MainActivity"
            help="The name of the activity class to create"/>

    <parameter
            id="layoutName"
            name="Layout Name"
            type="string"
            constraints="layout|unique|nonempty"
            suggest="${activityToLayout(activityClass)}"
            default="activity_main"
            help="The name of the layout to create for the activity"/>
    <parameter
            id="itemLayoutName"
            name="Item Layout Name"
            type="string"
            constraints="layout|unique|nonempty"
            suggest="${classToResource(activityClass)}_recycle_item"
            default="main_recycle_item"
            help="The name of the layout to create for the activity"/>
    <parameter
        id="itemCount"
        name="Item Count"
        type="int"
        default="15"/>
    <parameter
        id="listType"
        name="List Type"
        default="linear"
        type="enum">
        <option id="linear">Linear Layout</option>
        <option id="grid">Grid Layout</option>
        <option id="stagge">Staggered Grid Layout</option>
    </parameter>
    <parameter
        id="isSplit"
        name="Split Line"
        type="boolean"
        default="true"/>
    <parameter
        id = "packageName"
        name="Package Name"
        type="String"
        constraints="package"
        default="cn.sysmaster.template"/>
    <!-- 128x128 thumbnails relative to template.xml -->
    <thumbs>
        <!-- default thumbnail is required -->
        <thumb>template_blank_activity.png</thumb>
    </thumbs>

    <globals file="globals.xml.ftl"/>
    <execute file="recipe.xml.ftl"/>

</template>

```
### 2.2、用到的java类
SampleActivity.java.ftl
```java
package ${packageName};

import android.os.Bundle;

import com.chad.library.adapter.base.BaseQuickAdapter;
import com.chad.library.adapter.base.BaseViewHolder;

import java.util.ArrayList;
import java.util.List;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.recyclerview.widget.DividerItemDecoration;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;


public class ${activityClass} extends AppCompatActivity {

    private BaseQuickAdapter<String, BaseViewHolder> mAdapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.${layoutName});

        initView();
    }

    private void initView() {
        RecyclerView recyclerView = findViewById(R.id.recycler_view);
        <#switch listType>
            <#case 'linear'>
                recyclerView.setLayoutManager(new LinearLayoutManager(this));
            <#break>
            <#case 'grid'>
                recyclerView.setLayoutManager(new GridLayoutManager(this, 3));
            <#break>
            <#case 'stagge'>
                recyclerView.setLayoutManager(new StaggeredGridLayoutManager(3, StaggeredGridLayoutManager.VERTICAL));
            <#break>
        </#switch>
        <#if isSplit>
            DividerItemDecoration decoration = new DividerItemDecoration(this, DividerItemDecoration.VERTICAL);
            recyclerView.addItemDecoration(decoration);
        </#if>
        recyclerView.setAdapter(mAdapter = new BaseQuickAdapter<String, BaseViewHolder>(R.layout.${itemLayoutName}) {
            @Override
            protected void convert(@NonNull BaseViewHolder helper, String item) {
                helper.setText(R.id.text, item);
            }
        });
        loadData();
    }
    
    private void loadData() {
        List<String> datas = new ArrayList<>();
        int itemCount = ${itemCount};
        for (int i = 1; i <= itemCount; i++) {
            datas.add("item" + i);
        }
        mAdapter.setNewData(datas);
    }

}
```
这里我们看到，通过template.xml获取的参数，配置了Activity的类名、布局名、还有item个数，并且用到了freemarker的switch、if语法，动态设置layoutManager，这玩意看的懂就行，和java类似。

### 2.3、用的到布局ftl

activity_sample.xml.ftl
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recycler_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</LinearLayout>
```
recycle_item.xml.ftl
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical">

    <TextView
        android:id="@+id/text"
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:gravity="center_vertical"
        android:paddingLeft="15dp"
        android:textSize="16sp"
        tools:text="TextView" />

</LinearLayout>
```
这里没什么动态的内容，都是最简单的控件。这里也可以像java.ftl使用动态参数

### 2.4、recipe.xml.ftl
```xml
<?xml version="1.0"?>
<recipe>
    <!--1-->
    <dependency mavenUrl="androidx.recyclerview:recyclerview:+" />
    <dependency mavenUrl="com.github.CymChad:BaseRecyclerViewAdapterHelper:2.9.47-androidx" />
    <!--2-->
    <merge from="AndroidManifest.xml.ftl"
                 to="${escapeXmlAttribute(manifestOut)}/AndroidManifest.xml" />
    <!--3-->
    <instantiate from="src/app_package/SampleActivity.java.ftl"
                   to="${escapeXmlAttribute(srcOut)}/${activityClass}.java" />
    <instantiate from="res/layout/activity_sample.xml.ftl"
                   to="${escapeXmlAttribute(resOut)}/layout/${layoutName}.xml" />
    <instantiate from="res/layout/recycle_item.xml.ftl"
                   to="${escapeXmlAttribute(resOut)}/layout/${itemLayoutName}.xml" />
    <instantiate from="res/layout/recycle_item.xml.ftl"
                   to="${escapeXmlAttribute(resOut)}/layout/${itemLayoutName}.xml" />
    <!--4-->
    <open file="${escapeXmlAttribute(srcOut)}/${activityClass}.java" />
    <open file="${escapeXmlAttribute(resOut)}/layout/${layoutName}.xml" />
    <open file="${escapeXmlAttribute(resOut)}/layout/${itemLayoutName}.xml" />
</recipe>

```
由于globals.xml.ftl没添加什么变量，就不展开写了。recipe.xml.ftl定义的东西比较关键，我在上面注释分了四部分
1. 引入所需要的类库
2. 合并manifest文件
3. ftl->java,layout
4. 打开创建好的文件

其实这些已经在模板部分讲过了

### 参考链接
[https://blog.csdn.net/lmj623565791/article/details/51635533](https://blog.csdn.net/lmj623565791/article/details/51635533)

[https://blog.csdn.net/Marine_snow/article/details/79893509](https://blog.csdn.net/Marine_snow/article/details/79893509)

[https://www.jianshu.com/p/329c87049750](https://www.jianshu.com/p/329c87049750)


