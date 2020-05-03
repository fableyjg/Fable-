2020-03-29 fable说 Fable说

4月，苹果将禁用UIWebView；新提审的应用，将不再被审核；老的应用可以持续到12月；建议提前做好应对措施。

UIWebView是什么坑？若提审时，收到这样的邮件

关键信息：ITMS-90809: Deprecated API Usage — Apple will stop accepting submissions of apps that use UIWebView APIs

那么就代表应用中包含了UIWebView；这个API，从2020年4月起，苹果将限制。具体限制如文章开篇所述。

如何检查，自己的应用是否包含UIWebView呢？

1.最简单的方法，提审时，开发者邮箱是否收到这样的邮件。

2.如何在提审前，自测呢？或者说，如何定位是，哪一家的framework导致的呢？

这里提供stackoverflow上的一个答案，笔者就是通过这个方法，链接：https://stackoverflow.com/questions/57722616/itms-90809-deprecated-api-usage-apple-will-stop-accepting-submissions-of-app/57730461#57730461

2个命令，稍加解释

cd ~/Library/Developer/Xcode/Archives/<date>/myapp.xcarchive/Products/Applications/myapp.app
在终端中，找到对应Archive的版本，注意日期，找到对应app

然后通过

```
nm myapp | grep UIWeb

for framework inFrameworks/*.framework; do

  fname=$(basename $framework .framework)
  echo $fname
  nm $framework/$fname | grep UIWeb

done
```

这段代码遍历查找framework中是否包含有UIWeb。

如果查找的结果是：

error: Frameworks/*.framework/*: No such file or directory.

那么就代表没有。

如果查找结果有framewrok，加一堆16进制编码，顺着查找，应该能找到具体对应的framework。

更新这家的framework，就能解决。

ps：最近就是这个问题，导致一直没更新文章；因为需要升级mac系统，升级xcode，升级framewrok；后面尽量更新勤一点。感谢您的不离不弃。
