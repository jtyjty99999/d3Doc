>Learning D3 是本站准备推出的原创+翻译教程~作者主要是俺与各位d3大牛们。。。

>数据可视化是个热门话题，而D3.js则是前端数据可视化的超级工具。俺作为一个前端，有志于推动D3.js在中国的发展。。

>但是中文网络上关于D3.js的资料比较少，因此本系列教程将会以翻译外文D3教程及我个人撰写的形式来进行。不过虽然是翻译，但肯定不会照着抄，我会加入很多自己的理解。也希望大家学习的时候，多看d3.js的api与实例  https://github.com/mbostock/d3/wiki/API-Reference

>然后呢，俺翻译完后，会注明各种链接。俺的原创文章若有参考，也会注明链接。也同样请同行们注意

>*所有文章都欢迎转载分享，但是请注明好转载链接。不要因为自己一些个人的小利益，而影响整个行业圈的前行，所有的站点都应该是为整个行业的前行而贡献自己的热量，把更多的时间用于创造上*


对于我来说，当初让我决心学习D3的原因，不是D3.js的各种图表，因为市面上成熟的js图表库实在是太多，D3js最吸引我的是各种动态的可视化效果及一些强大的图形算法。

今天接着上一篇教程Learning D3.js(1) 学习制作一个柱形图/直方图  继续来讲解图表的绘制，不同的是，我们将引入动态的方式来展现图表。

##demo:

 [http://www.d3js.cn/demo/bar5.html](http://www.d3js.cn/demo/bar5.html "demo1")
 [http://www.d3js.cn/demo/bar6.html](http://www.d3js.cn/demo/bar6.html "demo2")

原文   [http://mbostock.github.com/d3/tutorial/bar-2.html](http://mbostock.github.com/d3/tutorial/bar-2.html "第二篇") 因为会涉及一些javascript知识，我也会在此补充。

首先为了实现“动态”我们需要引入时间。而javascript控制计时器的函数有两个
    
    setTimeout( foo, 1000 )//定义之后的一秒钟后会调用foo
    
    setInterval( foo, 1000 )//定义之后，每隔一秒钟都会调用foo。
理想状态下会实现以上的效果，我们可以配合计时器，每隔一秒钟刷新数据，之后根据数据来进行图表绘制即可。

但这里补充一点，因为javascript是单线程的，实际应用时，由于setTimeout的机制是会将foo这个函数加入到队列尾端再执行，因此若setTimeout定义的函数后面，执行函数时间过长，那么foo函数就不会在1秒后触发，而是超过一秒。比如这样的代码
    
    setTimeout( foo, 1000 )
    
    doSomethingTough()//耗时两秒
而setInterval则更加诡异，他会每隔一段时间，向队列里插入函数，这样如果一个函数执行时间过长，就会发现，执行完毕后，setInterval已经“积攒”了很多的foo没执行，就会同时执行很多- -

更为先进的实现是requtestAnimationFrame这个api。上面只是一段背景知识介绍，感兴趣的朋友可以继续了解下。

那么我们回到d3js的话题。首先我们利用耳熟能详的两个方法构建我们的图表基本结构

    var w = 20,
        h = 80;
    
    var x = d3.scale.linear()
        .domain([0, 1])
        .range([0, w]);
    
    var y = d3.scale.linear()
        .domain([0, 100])
        .rangeRound([0, h]);
        
大家回忆下上篇文章讲过的东西，x,y均是动态的对象，可以把数据转换为我们想要的宽度，高度，这里利用domain方法来设置需要匹配的数值范围，之后利用range来规定容器的宽度。其实大家仔细看，这里是一个小技巧。 domain里面是0,1 对应range是0,w 那么x(n)其实就等于n*w (因为1就是w么)。

所以这里的w的作用是规定一条数据所占的宽度

这里的rangeRound跟range很类似的，但提供了四舍五入功能。他的作用是为了让图表更加的平滑，之后我们进行动画操作的时候不至于显得很突兀。

之后我们就可以建立图表容器

    var chart = d3.select("body").append("svg")
        .attr("class", "chart")
        .attr("width", w * data.length - 1)
        .attr("height", h);
        
*注意观察，width就是数据的总长度*。

然后利用data方法给图表增加数据

    chart.selectAll("rect")
        .data(data)
      .enter().append("rect")
        .attr("x", function(d, i) { return x(i) - .5; })
        .attr("y", function(d) { return h - y(d) - .5; })
        .attr("width", w)
        .attr("height", function(d) { return y(d); });
        
回想下之前讲的，我们利用selectAll创建一个空容器，之后绑定数据，“进入”容器，然后把容器内放入一个矩形。

这里可能很多人会对下面的属性设置不太理解，我画张图大家就明白了~

![pic1](http://ww2.sinaimg.cn/bmiddle/409d840etw1e32hrf7n9zj.jpg)

一定要明确这几个属性的意义跟区别: x,y为横纵坐标，width,height为长宽，而且注意，坐标原点在左上角。

然后给图表应用样式

    .chart rect {
      fill: steelblue;
      stroke: white;
    }
图表效果如下：测试数据 :

    var data = [4, 8, 15, 16, 23, 42, 8, 15, 16, 23, 42, 8, 15, 16, 23, 42, 8, 15, 16, 23, 42]
![pic2](http://ww1.sinaimg.cn/bmiddle/409d840etw1e32ykocldaj.jpg)

之后我们让图表动起来。首先要建立动态的数据。

    setInterval(function(){
        data.push(data.shift())
        console.log(data)
    },1000)
在javascript中，数组的shift()方法可以把数组第一项“弹出来”并返回，而push方法则会在末尾加入元素。

之后我们使用定时器去循环这个操作，就得到了一系列的动态数据：

    [8, 15, 16, 23, 42, 4, 8, 15, 16, 23, 42, 8, 15, 16, 23, 42, 8, 15, 16, 23, 42]
    [15, 16, 23, 42, 4, 8, 15, 16, 23, 42, 8, 15, 16, 23, 42, 8, 15, 16, 23, 42, 8]
    [16, 23, 42, 4, 8, 15, 16, 23, 42, 8, 15, 16, 23, 42, 8, 15, 16, 23, 42, 8, 15]
    [23, 42, 4, 8, 15, 16, 23, 42, 8, 15, 16, 23, 42, 8, 15, 16, 23, 42, 8, 15, 16]
    [42, 4, 8, 15, 16, 23, 42, 8, 15, 16, 23, 42, 8, 15, 16, 23, 42, 8, 15, 16, 23]
动态图标的核心除了数据，就是变换。我们加入d3.js强大的变换工具。

思路，当数据改变时，重新绘制数据表即可

    function redraw1() {
        chart.selectAll("rect")
                .data(data)
                .transition()
                .duration(1000)
                .attr("y", function(d) { return h - y(d) - .5; })
                .attr("height", function(d) { return y(d); });
    }
    setInterval(function(){
        data.push(data.shift())
        console.log(data)
        redraw1()
    },1000)
redraw1函数重新选择矩形rect,将矩形绑定到新数据,然后开始一个过渡,更新“y”和“高度”属性。数据跟节点的绑定是通过index序号来绑定的。之后的demo效果已经很不错了，见

http://www.d3js.cn/demo/bar5.html

这时候很多人会担心这个问题，若数据在某个时刻停止传输，怎么办？数据的总数会不断减少啊，何来绑定？

同时很多人更会问  上面的图表有啥实际价值啊。。。上上下下的。。。

那么我们要认清变换的本质，是数据。刚才我们每个矩形绑定的是数据在数组中的index，这次我们要真正绑定具体的某条数据。

通常我们使用的数据是这样的

[{value:123,time:1364269017893},{value:88,time:1364269037338}]之类。
这类数据的特点就是每个点代表每个时刻，每条数据带着自己的时间戳。此时我们如果对矩形绑定数据的时间戳，就可以实时跟踪这条数据了。

我们用下面的方法生成一系列测试数据

    var timer = setInterval(function () {
        if (count == 20) {
            clearInterval(timer)
            chartInit()
        }
        data.push({
            'value' : parseInt(100 * Math.random()),
            'time' : +new Date()
        })
        count += 1
    }, 10)
简单解释一下，Math.random可以返回0-1的随机数，*100再用parseInt取整，表示1-100的随机数。而time我们取当前时间。+new Date()方法可以通过javascript取到当前时间的毫秒数。当数据超过20条时，停止生成测试数据(clearInterval方法把计时器停掉，同时执行绘制图表函数)

之后根据数据，我们制作一个X轴变换的方法。

     function redraw() {
    
                var rect = chart.selectAll("rect")
                        .data(data, function (d) {
                            return d.time;
                        });
    
                // Enter…
                rect.enter().insert("rect", "line")
                        .attr("x", function (d, i) {
                            return x(i) - .5;
                        })
                        .attr("y", function (d) {
                            return h - y(d.value) - .5;
                        })
                        .attr("width", w)
                        .attr("height", function (d) {
                            return y(d.value);
                        });
    
                // Update…
                rect.transition()
                        .duration(1000)
                        .attr("x", function (d, i) {
                            return x(i) - .5;
                        });
    
                // Exit…
                rect.exit()
                        .remove();
            }
这时候在绑定rect的时候，就使用data.time时间戳做的绑定。这样我们可以跟踪某条数据，并对这条数据进行变换。

这里我们可以看到使用了  transition方法，对数据的x轴坐标进行变换。

变换完毕后，因为不断有新数据进来，我们要删掉原有的矩形，使用remove()方法即可。若不使用remove方法，就会看到左边堆积了一大堆历史数据。。
[http://www.d3js.cn/demo/bar6.html](http://www.d3js.cn/demo/bar6.html "demo2")