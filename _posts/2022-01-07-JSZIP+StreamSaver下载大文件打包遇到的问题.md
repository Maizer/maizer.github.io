---
title: JSZIP+StreamSaver下载大文件打包遇到的问题
tags: Code
---

背景:公司项目为了节约服务器硬盘,需要从微信企业微盘下载大批量文件,然后,进行客户端打包.

问题:

根据StreamSaver的代码演示例子,是通过指定Blob类型进行流下载更新,我尝试了这个方法,发现一旦Blob类型大于1G的时候,Chrome浏览器便会抛出Type Error:network error.无法进行下载.也就是说当文件批量下载缓存到内存中后,JSZIP工作正常,可以进行压缩,但是,一旦执行reader.read()方法便会抛出异常.

解决:查阅JSZIP的代码与StreamSaver的代码发现,其对uint8array编码支持更好,更换成uint8array编码之后,问题解决,即便打包2G甚至更大的文件也不会出现错误.     

总结:其原因在于由于示例代码的误导,走了不少弯路   
```
  Promise.all(promises).then(() => {


                zip.generateAsync({
                    type: 'uint8array',
                    streamFiles: true,
                    compression: 'DEFLATE',
                    compressionOptions: {
                        level: 9
                    }
                }, function updateCallback(metadata) {

                    let v = metadata.percent.toFixed(0);
                    $("#progressbar_id").attr("aria-valuenow", v);

                }).then(content => {

                    let max = content.length;

                    window.fileStream = streamSaver.createWriteStream(filename + '.zip', {
                        size: max
                    });
                    window.writer = fileStream.getWriter();
                    window.onunload = () => writer.abort();

                    writer.write(content).then(function () {
                        writer.close();
                        window.onunload = null;
                        window.writer = null;
                        window.fileStream = null;
                        content = null;
                    });

                });
            }).catch(res => {
                console.log("压缩失败");
            });
```
