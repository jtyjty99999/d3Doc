今天算是复习教程，复习一下之前学到的transition变换效果，及d3.scale.linear(),domain(),range()几个方法。

不过我们会使用一个新的api:axis

demo
 [http://www.d3js.cn/demo/bar7.html](http://www.d3js.cn/demo/bar7.html "demo1")
 [http://www.d3js.cn/demo/bar8.html](http://www.d3js.cn/demo/bar8.html "demo2")


首先这是一个散点图 那么我们需要在绘制的时候，使用svg的circle标签绘制圆形。

	<svg width="50" height="50">
	<circle cx="25" cy="25" r="25" fill="purple" />
	</svg>
注意circle标签的属性 cx表示圆心横坐标，cy表示圆心纵坐标，r表示圆心半径

我们的数据是这样的

    var dataset = [
        [5, 20,1], [480, 90,2], [250, 50,3], [100, 33,4], [330, 95,5],
        [410, 12,6], [475, 44,7], [25, 67,8], [85, 21,9], [220, 88,10],
        [600, 150,11]
    ];
首先仍然要建立一个容器

    //Width and height
    var w = 600;
    var h = 400;
    var padding = 20;
    var chart = d3.select("body")
            .append("svg")
            .attr("width", w)
            .attr("height", h);
之后呢，利用d3.scale.linear()，配合domain数据，及range范围，规定图表内点的属性。

    var xScale = d3.scale.linear()
            .domain([0, d3.max(dataset, function(d) { return d[0]; })])
            .range([padding, w - padding * 2]);//x轴

    var yScale = d3.scale.linear()
            .domain([0, d3.max(dataset, function(d) { return d[1]; })])
            .range([h - padding, padding]);//y轴

    var rScale = d3.scale.linear()
            .domain([0, d3.max(dataset, function(d) { return d[1]; })])
            .range([5, 15]);//半径，5-15
利用selectAll配合data方法切分图表

    chart.selectAll("circle")
            .data(dataset)
            .enter()
            .append('circle')
            .attr("cx", function(d) {
                return xScale(d[0]);
            })
            .attr("cy", function(d) {
                return yScale(d[1]);
            })
            .attr("r", function(d) {
                return rScale(d[1]);
            });
这时候，横坐标就是我们数据中数组第一项，而纵坐标就是第二项，半径在这里为了方便，使用第二项作为基准。

这样图表的效果基本ok

![pic1](http://www.d3js.cn/wp-content/uploads/2013/04/快照3.png)

然后我们让图表动起来。

    setInterval(function(){
        dataset.push(dataset.shift())
        redraw()
    },1000)
利用setInterval计时器，更新数据，每隔一秒，把数据的第一项扔到最后，之后redraw则是重绘图表的过程。

    function redraw() {
        chart.selectAll("circle")
                .data(dataset)
                .transition()
                .duration(1000)
                .attr("cy", function(d) {
                    return yScale(d[1]);
                })
                .attr("r", function(d) {
                    return rScale(d[1]);
                })
    }
那么第一个demo就ok了  [http://www.d3js.cn/demo/bar7.html](http://www.d3js.cn/demo/bar7.html "demo1")

这样的图表太单调了，没有横轴纵轴的标识。之前我们使用line标签来画线，而d3.js提供了方便的api来绘制这类坐标轴。

一般比较常用的是 axis.scale([scale])与axis.orient([orientation])两个方法。

前者与d3.scale.linear类似，可以绑定数据，而后者可以规定轴的方向（横轴或者纵轴）。当用户的值设置为top或者bottom时，为水平轴，而left或者right时，为垂直轴。

axis生成后，包括line或者path标签，还有text标签，可以方便应用样式。

使用axis时一般这样用：

    var xAxis = d3.svg.axis()
            .scale(xScale)
            .orient("bottom").ticks(5)//首先定义一个轴。tick跟之前讲过的tick类似，是规定刻度的间隔数

	chart.append("g")//创建axis的区域
		.attr("transform", "translate(0,30)")//进行位移
		.call(axis)//绑定预先定义好的axis
先定义一个轴，再用call方法绑定这个轴

这样我们给之前的图表加上横轴纵轴

    //Define x axis
    var xAxis = d3.svg.axis()
            .scale(xScale)
            .orient("bottom").ticks(5)

    //Create x axis
    chart.append("g")
            .attr("class", "axis")
            .attr("transform", "translate(0," + (h - padding) + ")")
            .call(xAxis);
    //Define Y axis
    var yAxis = d3.svg.axis()
            .scale(yScale)
            .orient("left").ticks(5)

	//Create Y axis
    chart.append("g")
                    .attr("class", "axis")
                    .attr("transform", "translate(30,0)")
                    .call(yAxis);
这里的translate方法接收两个参数，分别是水平位移与垂直位移。当然我们也可以配合class，来给坐标轴应用样式。

	.axis path,
	.axis line {
		fill: none;
		stroke: black;
		shape-rendering: crispEdges;
	}

	.axis text {
		font-family: sans-serif;
		font-size: 11px;
	}
加上坐标轴我们的图形就漂亮多了，效果如下
![pic2](http://www.d3js.cn/wp-content/uploads/2013/04/快照4.png)


demo见[http://www.d3js.cn/demo/bar8.html](http://www.d3js.cn/demo/bar8.html "demo2")

 