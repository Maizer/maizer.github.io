---
title: JSZIP+StreamSaver下载大文件打包遇到的问题
tags: Code
---

2022-01-07

背景:公司项目为了节约服务器硬盘,需要从微信企业微盘下载大批量文件,然后,进行客户端打包.

问题:

根据StreamSaver的代码演示例子,是通过指定Blob类型进行流下载更新,我尝试了这个方法,发现一旦Blob类型大于1G的时候,Chrome浏览器便会抛出Type Error:network error.无法进行下载.也就是说当文件批量下载缓存到内存中后,JSZIP工作正常,可以进行压缩,但是,一旦执行reader.read()方法便会抛出异常.

解决:查阅JSZIP的代码与StreamSaver的代码发现,其对uint8array编码支持更好,更换成uint8array编码之后,问题解决,即便打包2G甚至更大的文件也不会出现错误.     

总结:其原因在于由于示例代码的误导,走了不少弯路   
```javascript
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
2023-04-06

我对于上一次中支持更好的答案其实并不是肯定的,因为终究没有理解其中的原理.
对此我这两天一直在反复的查询相关资料.以试图解开Blob与Uint8Array的区别.

https://stackoverflow.com/questions/17823225/do-arraybuffers-have-a-maximum-length

我在这里发现了Uinit8Array的一个答案,浏览器对ArrayBuffer是没有施加限制的,理论上只与用户的内存大小相关.

https://stackoverflow.com/questions/42614880/download-large-data-stream-1gb-using-javascript

我在这里发现了Blob的答案,浏览器会对Blob施加大小的限制.

初步来看,应该是浏览器对两种类型的内部处理是不同的.
我有种可能不是标准的想法:
ArrayBuffer在内部进行了Blob分块处理,而Blob需要手动进行分块处理.所以ArrayBuffer不会抛出异常,而Blob会出现异常,可能因为每个Blob块都有大小限制.

到此虽然离最终答案更近一步,但是我依然还是不太理解,我可能需要追查更多的资料,去理解浏览器内部到底如何对这两种类型进行处理的.
