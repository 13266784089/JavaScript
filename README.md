# JavaScript
<!DOCTYPE html>
<html>
<head>
	<title>JavaScript Loan Calculator</title>
	<style type="text/css">/* 这是一个css样式表：定义了程序输出的样式*/ 
		.output {font-weight: bold;}            /*计算结果定义为粗体*/       
		#payment {text-decoration: underline;}  /*定义 id="payment" 的元素样式*/  
		#graph {border: solid black 1px;}       /*图表有一个1像素的边框*/
		th,td {vertical-align: top;}            /*表格单元格对其方式为顶端对齐*/
	</style>
</head>
<body>
<!-- 只是一个HTML表格，其中包含<input>元素可以用来输入数据。
     程序将在<span>元素中显示计算结果，这些元素都具有类似"interset"和"years"的id
     这些id将在表格下面的JavaScript代码中用到。我们注意到，有一些input元素定义了
     "onchange"或"onclick"的事件处理程序，以便用户在输入数据或点击inputs时执行指
     定的JavaSCript代码段
     -->
<table>
	<tr>
		<th>Enter Loan Data:</th>
		<td></td>
		<th>Loan Balance,Cumulative Equity,and Interest Payments</th>
	</tr>
	<tr>
		<td>Amount of the loan ($):</td>
		<td><input id="amount" onchange="calculate();"></td>
		<td rowspan="8"><canvas id="graph" width="400px" height="250px"></canvas></td>
	</tr>
	<tr>
		<td>Annual interset (%):</td>
		<td><input id="apr" onchange="calculate();"></td>
	</tr>
	<tr>
		<td>Repayment period (years):</td>
		<td><input id="years" onchange="calculate();"></td>
	</tr>
	<tr>
		<td>Zipcode (to find lenders):</td>
		<td><input id="zipcode" onchange="calculate();"></td>
	</tr>
	<tr>
		<th>Approximate Payments:</th>
		<td><button onclick="calculate();">Calculate</button></td>
	</tr>
	<tr>
		<td>Monthly payment:</td>
		<td>$<span class="output" id="payment"></span></td>
	</tr>
	<tr>
		<td>Total payment:</td>
		<td>$<span class="output" id="total"></span></td>
	</tr>
	<tr>
		<td>Total interest:</td>
		<td>$<span class="output" id="totalinterest"></span></td>
	</tr>
	<tr>
		<th>Sponsors:</th>
		<td colspan="2">
			Apply for your loan with one of these fine lenders:
			<div id="lenders"></div>
		</td>
	</tr>
</table>

<!-- 随后是JavaScript代码，这些代码内嵌在了一个<script>标签里 -->
<!-- 通常情况下，这些脚本代码应当放在<head>标签中 -->
<!-- 将JavaScript代码放在HTML代码之后仅仅是为了便于理解 -->
<script type="text/javascript">
"use strict";  //如果浏览器支持的话，则开启ECMAScript 5的严格模式
/*这里的脚本定义了caculate()函数，在HTML代码中绑定事件处理程序时会调用它
*这个函数从<input>元素中读取数据，计算贷款赔付信息，并将结果显示在<span>元素中
*同样，这里还保存了用户数据，展示了放贷人链接并绘制出了图表
*/
function calculate(){
	//查找文档中用于输入输出的元素
	var amount = document.getElementById("amount");
	var apr = document.getElementById("apr");
	var years = document.getElementById("years");
	var zipcode = document.getElementById("zipcode");
	var payment = document.getElementById("payment");
	var total = document.getElementById("total");
	var totalinterest = document.getElementById("totalinterest");

	//假设所有的输入都是合法的，将在input元素中获取输入数据
	//将百分比格式转换为小数格式，并从年利率转换为月利率
	//将年度赔付转换为月度赔付
	var principal = parseFloat(amount.value);
	var interest = parseFloat(apr.value) / 100 / 12;
	var payments = parseFloat(years.value) * 12;

	//现在计算月度赔付的数据
	var x = Math.pow(1 + interest,payments);  //Math.pow() 进行幂次运算
	var monthly = (principal * x * interest) / (x - 1);

	//如果结果没有超过JavaScript能表示的数字范围，且用户的输入也正确
	//这里所展示的结果就是合法的
	if(isFinite(monthly)) {
		//将数据填充至输出字段的位置，四舍五入到小数点后两位数字
		payment.innerHTML = monthly.toFixed(2);
		total.innerHTML = (monthly * payments).toFixed(2);
		totalinterest.innerHTML = ((monthly * payments) - principal).toFixed(2);

		//将用户的输入数据保存下来，这样在下次访问时也能取到数据
		save(amount.value,apr.value,years.value,zipcode.value);

		//找到并展示本地放贷人，但忽略网络错误
		try { //捕获这段代码抛出的所有异常
			getLeaders(amount.value,apr.value,years.value,zipcode.value);
		}
	    catch(e){/*忽略这些异常*/}

	    //最后，用图表展示贷款余额，利息和资产收益
	    chart(principal,interest,monthly,payments);
	} else {
		//计算结果不是数字或者无穷大，意味着输入数据是非法或不完整的
		//清空之前的输出数据
		payment.innerHTML = "";
		total.innerHTML = "";
		totalinterest.innerHTML = "";
		chart();
	}
}
//将用户的输入保存至localStorage对象的属性中，这些属性在再次访问时还会继续保持在原位置
//如果你在浏览器中按照file://URL的方式直接打开本地文件，则无法在某些浏览器中使用存储功能
//而通过HTTP打开文件是可行的
function save(amount,apr,years,zipcode){
	if (window.localStorage) {
        localStorage.loan_amount = amount;
        localStorage.loan_apr = apr;
        localStorage.loan_years = years;
        localStorage.loan_zipcode =	zipcode;	 
	}
}

// 在文档首次加载时，将会尝试还原输入字段
window.onload = function(){
	//如果浏览器支持本地存储并且上次保存的值是存在的
	if(window.localStorage && localStorage.loan_amount){
		document.getElementById("amount").value = localStorage.loan_amount;
		document.getElementById("apr").value = localStorage.loan_apr;
		document.getElementById("years").value = localStorage.loan_years;
		document.getElementById("zipcode").value = localStorage.loan_zipcode;
	}
};

//将用户的输入发送至服务器端脚本（理论上）将返回一个本地放贷人的链接列表，
//在这个例子中并没有实现这种查找放贷人的服务
//但如果改服务器存在，该函数会使用它
function getLenders(amount,apr,years,zipcode){
	//如果浏览器不支持XMLHttpRequest对象，则退出
	if (!window.XMLHttpRequest) return;

	//找到要显示放贷人列表的元素
	var ad = document.getElementById("lenders");
	if(!ad) return;              //如果返回为空，则退出  

	//将用户的输入数据进行URL编码，并作为查询参数附加在URL里
	var url = "getLenders.php"+            //处理数据的URL地址
	"?amt=" + encodeURLComponent(amount) + //使用查询串中的数据
	"&apr=" + encodeURLComponent(apr) +
	"&yrs=" + encodeURLComponent(years) +
	"&zip=" + encodeURLComponent(zipcode);

	//通过XMLHttpRequest对象来提取返回数据
	var req = new XMLHttpRequest();         //发起一个新的请求
	req.open("GET",url);					//通过URL发起一个HTTP GET请求
	req.send(null);							//不带任何正文发送这个请求

	//在返回数据之前，注册了一个事件处理函数，这个处理函数将会在服务器的响应
	//返回至客户端的时候调用
	//这种异步编程模型在客户端JavaScript中非常常见的
	req.onreadystatechange = function(){
		if (req.readyState == 4 && req.status == 200) {
			//如果代码运行到这里，说明我们得到了一个合法且完整的HTTP响应
			var response = req.responseText;      //HTTP响应是以字符串的形式呈现的  
			var lenders = JSON.parse(response);   //将其解析为JS数组

			// 将数组中的放贷人对象转换为HTML字符串形式
			var list = "";
			for (var i = 0 ; i < lenders.length; i++){
				list += "<li><a href='"+ lenders[i].url +"'>" +
						lenders[i].name + "</a>";
			}

			//将数据在HTML元素中呈现出来
			ad.innerHTML = "<ul>" + list + "</ul>";
		}
	}
}

//在HTML<canvas>元素中用图表展示月度贷款余额，利息和资产收益
//如果不传入参数的话，则清空之前的图表数据
function chart(principal,interest,monthly,payments){
	var graph = document.getElementById("graph");  //得到<canvas>标签
	graph.width = graph.width; //用一种巧妙的手法清除并重置画布

	//如果不传入参数，或者浏览器不支持画布，则直接返回
	if (arguments.length == 0 || !graph.getContext) return;

	//获得画布元素的"context"对象，这个对象定义了一组绘画API
	var g = graph.getContext("2d");              //所有的绘画操作都将基于这个对象
	var width = graph.width,
	height = graph.height;						 //获得画布大小	

	//这里的函数作用是将付款数字和美元数据转换为像素
	function paymentToX(n){
		return n * width / payments;
	}
	function amountToY(a){
		return height - (a *height /(monthly * payments *1.05));
	}

	//付款数据是一条从（0，0）到（payments，monthly*payments）的直线
	g.moveTo(paymentToX(0),amountToY(0));
	g.lineTo(paymentToX(payments),amountToY(monthly*payments));
	g.lineTo(paymentToX(payments),amountToY(0));
	g.closePath();
	g.fillStyle = "#f88";
	g.fill();
	g.font = "bold 12px sans-serif";
	g.fillText("Total Interest Payments",20,20);

	//很多资产数据并不是线性的，很难将其反映至图表中
	var equity = 0;
	g.beginPath();
	g.moveTo(paymentToX(0),amountToY(0));
	for(var p = 1; p <= payments; p++){
		var thisMonthsInterest = (principal - equity) * interest;
		equity += (monthly - thisMonthsInterest);
		g.lineTo(paymentToX(p),amountToY(equity));
	}
	g.lineTo(paymentToX(payments),amountToY(0));
	g.closePath();
	g.fillStyle = "green";
	g.fill();
	g.fillText("Total Equity",20,35);

	//再次循环，余额数据显示为黑色粗线条
	var bal = principal;
	g.beginPath();
	g.moveTo(paymentToX(0),amountToY(bal));
	for (var p = 1; p <= payments; p++){
		var thisMonthsInterest = bal * interest;
		bal -= (monthly - thisMonthsInterest);
		g.lineTo(paymentToX(p),amountToY(bal));
	}
    g.lineWidth = 3;
    g.stroke();
    g.fillStyle = "black";
    g.fillText("Loan Balance",20,50);

    //将年度数据在X轴做标记
    g.textAlign = "center";
    var y = amountToY(0);
    for(var year = 1; year * 12 <= payments; year++){
    	var x = paymentToX(year * 12);
    	g.fillRect(x - 0.5, y - 3 , 1 , 3);
    	if (year == 1) g.fillText ("year", x, y - 5);
    	if (year % 5 == 0 && year * 12 !== payments) g.fillText(String(year), x, y - 5);
    }

    //将赔付数额标记在右边界
    g.textAlign = "right";
    g.textBaseline = "middle";
    var ticks = [monthly * payments, principal];
    var rightEdge = paymentToX(payments);
    for (var i = 0; i < ticks.length; i++){
    	var y = amountToY(ticks[i]);
    	g.fillRect(rightEdge - 3, y - 0.5, 3 ,1)
    	g.fillText(String(ticks[i].toFixed(0)),rightEdge - 5, y);
    }
}
</script>
</body>
</html>
