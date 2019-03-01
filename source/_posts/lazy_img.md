---
title: 图片延迟加载
categories:  js
tags: 
- js
---

## js实现图片延迟加载

###  图片延迟加载的目的
> 延迟加载图片，解决首屏快速加载的问题，网页打开的速度是第一要素,加载html时会去解析img的src,http请求下载图片，会阻塞页面加载。主要目的是作为服务器前端的优化，减少请求数或延迟请求数，对服务器缓压

###  延迟加载的实现

Web图片的懒加载就是通过读取img元素，然后获得img元素的origin-src（也可以约定为其他属性名）属性的值，并赋予img的src，从而实现动态加载图片的机制

#### HTML约定
```javascript
 <div class="container">
        <div class="img-area"><img src="" alt="loading" class="my-photo lazyimg" data-original="http://axuebin.com/lazyload/img/img1.png"></div>
        <div class="img-area"><img src="" alt="loading" class="my-photo lazyimg" data-original="http://axuebin.com/lazyload/img/img2.png"></div>
        <div class="img-area"><img src="" alt="loading" class="my-photo lazyimg" data-original="http://axuebin.com/lazyload/img/img3.png"></div>
        <div class="img-area"><img src="" alt="loading" class="my-photo lazyimg" data-original="http://axuebin.com/lazyload/img/img4.png"></div>
        <div class="img-area"><img src="" alt="loading" class="my-photo lazyimg" data-original="http://axuebin.com/lazyload/img/img5.png"></div>
        <div class="img-area"><img src="" alt="loading" class="my-photo lazyimg" data-original="http://axuebin.com/lazyload/img/img6.png"></div>
        <div class="img-area"><img src="" alt="loading" class="my-photo lazyimg" data-original="http://axuebin.com/lazyload/img/img7.png"></div>
        <div class="img-area"><img src="" alt="loading" class="my-photo lazyimg" data-original="http://axuebin.com/lazyload/img/img8.png"></div>
        <div class="img-area"><img src="" alt="loading" class="my-photo lazyimg" data-original="http://axuebin.com/lazyload/img/img9.png"></div>
        <div class="img-area"><img src="" alt="loading" class="my-photo lazyimg" data-original="http://axuebin.com/lazyload/img/img10.png"></div>
    </div>
```


### javascript实现
> data-src
```javascript
    window.onload = checkImgs
    window.onscroll = throttle(checkImgs)
    // 页面加载完成 检查一下图片，如果图片在第一屏，显示出来
    function checkImgs () {
        console.log(Array.from(imgs).entries())
        for (let [i,img] of Array.from(imgs).entries()) {
            // console.log(i)
            
            if (isInSight(img)) {
                loadImg(img)
            }
        }
    }

    // 滚动 浏览器是个窗口 viewport 对应的网页内容发生变化 scrollTop
    // 检测， 我们滚过的那块区域  是否有还未显示的图片
    function isInSight (el) {
        const bound = el.getBoundingClientRect()
        const clientHeight = window.innerHeight
        console.log(bound, clientHeight)
        // alert(bound.top);
        // alert(clientHeight);
        return bound.top <= clientHeight + 100
    }

     function loadImg (el) {
        const src = el.dataset.src
        if (src !== el.src) {
            const source = src
            // alert(source)
            let img = document.createElement('img')
            img.onload = function() {
                el.src = source
            }
            img.src = source
            
        }
    }

    // throttle 节流,不要执行的太频繁
    // 高阶函数的应用场景 增强了函数的功能
    function throttle (fn, mustRun = 500) {
        const timer = null
        let previous = null
        return function () {
            const now = new Date()
            const that = this
            const args = arguments
            if(!previous) {
                previous = now
            }
            const remaining = now - previous
            if (mustRun && remaining >= mustRun) {
                fn.apply(that, args)
                previous = now
            }
        }
    }

```
### jquery实现
> data-original
```javascript
    <script src="http://cdn.bootcss.com/jquery/2.1.0/jquery.min.js"></script>
    <script src="http://cdn.bootcss.com/jquery_lazyload/1.9.7/jquery.lazyload.min.js"></script> //顺序不可颠倒
    $(function() { 
        $(".lazyimg").lazyload({ 
            placeholder: '',
            effect : "fadeIn",
            threshold: 200
        }); 
    }); 
```