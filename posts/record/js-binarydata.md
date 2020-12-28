---
title: js 二进制数据操作
date: 2020-12-07 11:58:00
tags: [“nodejs”, “buffer”, “blob"]
categories: ["记录"]
---

> 无论是浏览器端还是服务器端，在处理文件或图片的时候都会用到以下几个对象方法：Buffer, ArrayBuffer, Blob, File, Base64 等等，这里总结上几个对象间是如何转换

## 链接

* [blob - mdn](https://developer.mozilla.org/en-US/docs/Web/API/Blob/Blob)
* [FileReader - mdn](https://developer.mozilla.org/en-US/docs/Web/API/FileReader)
* [URL.createObjectURL - mdn](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL)
* [atob/btoa - mdn](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/atob)
* [arraybuffer 文章](https://blog.techbridge.cc/2017/09/24/binary-data-manipulations-in-javascript/)
* [blob, arraybuffer, buffer 讲解](https://zhuanlan.zhihu.com/p/97768916)



![](http://md.rni-l.com/md/js-binarydata.png)



## 遇到的实际场景

### 文件（blob/file）生成可下载链接/文本/ArrayBuffer/BinaryString

利用 `FileReader` 这个构造函数，里面的 `readAsDataURL` 方法，将 `blob` 转成 `DataURL`

```javascript
function change(file) {
  const fileReader = new FileReader();
  fileReader.onload = (ev) => {
    if (!ev.target) return;
    setUrl(ev.target.result);
  };
  fileReader.readAsDataURL(file);
}

function download() {
  const a = document.createElement('a');
  a.download = fileName;
  a.href = url;
  a.click();
}
```

`readAsArrayBuffer`: blob -> arraybuffer

`readAsArrayText`: blob -> string

`readAsArrayBinaryString`: blob -> binary string

`readAsArrayDataURL`: blob -> url



### 接口获取到的 arraybuffer 转成 blob

使用 `Blob` 构造函数，传入即可

```javascript
const xhr = new XMLHttpRequest();
xhr.addEventListener('load', () => {
  if (xhr.readyState !== 4) return;
  const fileReader = new FileReader();
  fileReader.onload = (ev) => {
    if (!ev.target) return;
    console.log(ev.target.result) // url
  };
  const blob = new Blob([xhr.response]); // arraybuffer to blob
  fileReader.readAsDataURL(blob);
});
xhr.responseType = 'arraybuffer';
xhr.open('GET', remoteUrl);
xhr.send();
```



### base64/字符串 转 blob

base64 转 blob 其实就是字符串转 blob

只要生成字符串长度的 `uint8Array`，插入对应字符串的编码索引值，就可以生成 `blob`

```javascript
function base64ToBlob(code) {
  // 移除 base64 字符
  const parts = code.split(';base64,');
  const contentType = parts[0].split(':')[1];
  const raw = window.atob(parts[1]);
  const rawLength = raw.length;
  // 生成对应长度的 1 字节数组
  const uInt8Array = new Uint8Array(rawLength);
  // 将二进制数据转成对应的字节码索引
  for(let i = 0; i < rawLength; ++i) {
    uInt8Array[i] = raw.charCodeAt(i);
  }
  // 生成 blob
  return new Blob([ uInt8Array ], {
    type: contentType
  });
}
```



## Blob

`blob` 是由 `Blob` 构造函数生成的，通过传入的参数数组值排列组成的内容。我的理解，`blob` 是一个二进制数据处理对象，不同的数据类型：`ArrayBuffer`、`ArrayBufferView`、`USVString` 等等对象，传入生成的 `blbo`，再利用 `FileReader` 生成需要的数据格式。



## Buffer

## Base64(text)

