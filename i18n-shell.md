# I18N Shell

CakePHP的i18n功能使用[po文件](http://en.wikipedia.org/wiki/GNU_gettext)作为其翻译来源。PO文件与常用的翻译工具（如[Poedit）集成在一起](http://www.poedit.net/)。

i18n shell提供了一种快速简单的方式来生成po模板文件。然后可以将这些模板文件提供给翻译器，以便它们可以翻译应用程序中的字符串。完成翻译后，文件可以与现有翻译合并，以帮助您更新翻译。

## 生成POT文件

可以使用`extract`命令为现有应用程序生成POT文件。此命令将扫描整个应用程序的`__()`样式函数调用，并提取消息字符串。应用程序中的每个唯一字符串将被组合成一个单一的POT文件：

```
bin/cake i18n extract
```

以上将运行提取shell。此命令的结果将是文件**src / Locale / default.pot**。您可以使用pot文件作为创建po文件的模板。如果您正在从锅文件手动创建po文件，请确保正确设置`Plural-Forms`标题行。

### 生成插件的POT文件

您可以使用以下方式为特定插件生成POT文件：

```
bin/cake i18n extract --plugin <Plugin>
```

这将生成插件中使用的所需POT文件。

### 一次从多个文件夹中提取

有时，您可能需要从应用程序的多个目录中提取字符串。例如，如果要在应用`config`程序目录中定义一些字符串，则可能需要从该目录以及目录中提取字符串`src`。您可以使用该`--paths`选项来执行此操作。它需要以逗号分隔的绝对路径列表来提取：

```
bin/cake i18n extract --paths /var/www/app/config,/var/www/app/src
```

### 排除文件夹

您可以传递您希望被排除的文件夹的逗号分隔列表。包含带有提供值的路径段的任何路径将被忽略：

```
bin/cake i18n extract --exclude Test,Vendor
```

### 跳过覆盖现有POT文件的警告

通过添加`--overwrite`，如果POT文件已经存在并且默认情况下将覆盖，shell脚本将不会再提醒您：

```
bin/cake i18n extract --overwrite
```

### 从CakePHP核心库提取消息

默认情况下，提取shell脚本会询问您是否要提取CakePHP核心库中使用的消息。设置`--extract-core`为yes或no设置默认行为：

```
bin/cake i18n extract --extract-core yes

// or

bin/cake i18n extract --extract-core no
```



