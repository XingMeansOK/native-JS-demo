# addEventListener和attachEvent

标准浏览器中可以使用addEventListener()函数来给DOM元素绑定事件，使用removeEventListener()函数移除事件绑定，而IE6 IE7 IE8不支持addEventListener()和removeEventListener()，只能使用attachEvent()和detachEventListener()方法。IE从IE9开始支持addEventListener和removeEventListener.

addEventListener()和attachEvent()的区别主要有以下几点：

1. addEventListener(type,handler,capture)有三个参数，其中type是事件名称，如click，handler是事件处理函数，capture是否使用捕获，是一个布尔值，一般为false，这是默认值，所以第三个参数可以不写
attachEvent('on'+type,handler)有两个参数，其中type是事件名称，如click，第一个参数必须是onxxxx，handler是事件处理函数，IE6 IE7 IE8不支持事件捕获的，只支持事件冒泡。

> 关于事件冒泡：如果一个元素和它的各层上级素都设置了相同的事件，比如click事件，那么冒泡是指，当用户点击鼠标后，先触发最底层元素的click事件，然后依次逐层触发上一级元素的click事件，就像泡泡从水底冒到水面一样。而与事件冒泡相反，事件捕获的顺序是，从页面的根元素开始依次逐层向下级触发事件。IE6-8只支持事件冒泡不支持事件捕获，Google Chrome/Firefox/Safari/Opera等标准浏览器支持事件捕获和事件冒泡。如果只想在最底层执行事件处理函数，阻止向上级元素冒泡，也就是说如果要阻止事件冒泡，标准浏览器可以使用stopPropagation()，IE6 7 8可以使用cancelBubble=true

2. addEventListener绑定的事件是先绑定先执行，attachEvent绑定的事件是先绑定后执行

3. 在IE6 IE7 IE8浏览器中，使用了attachEvent或detachEvent后事件处理函数里面的this指向window对象，而不是事件对象元素，文末有解决方案

4. 如果一个事件在绑定以后有解除绑定的需要，可以使用removeEventListener()，参数和addEventListener一样，IE6 IE7 IE8使用detachEvent函数，参数和attachEvent一样，注意，**想要移除绑定的事件，则绑定事件时不能使用匿名函数**，而需要将事件处理函数单独写成一个函数。否则无法移除绑定的事件函数。匿名函数和匿名函数是互不相同，即使代码完全一样，但本质上同一匿名函数的代码写两次就是两个不同的函数。

```
<a id="xc">点击</a>
<script type="text/javascript">
        function handler(){
                alert(this.innerHTML);
        }
        var object=document.getElementById('xc');
        xc.addEventListener('click',handler,false);//添加事件
        xc.removeEventListener('click',handler,false);//移除事件
        /*下面这种使用匿名函数的方式添加的事件是不能移除的，因为两个匿名函数虽然代码一样，但本质上他们是两个不同的函数
        xc.addEventListener('click',function(){
                alert(this.innerHTML);
        },false);
        xc.removeEventListener('click',function(){
                alert(this.innerHTML);
        },false);
        */
</script>
```
5. 所添加的事件必须是事件对象支持的事件，比如给window对象添加click事件就不会成功，因为window没有click事件

6. 解决IE6 IE7 IE8 attchEvent this指向window的方法：
 1. 使用事件处理函数.apply(事件对象,arguments),以下代码仅适用于IE6 IE7 IE8这种方式的缺点是绑定的事件无法取消绑定，原因上面已经说了，匿名函数和匿名函数之间是互不相等的。
 ```
 <a id="xc">点击</a>
  <script type="text/javascript">
          var object=document.getElementById('xc');
          function handler(){
                  alert(this.innerHTML);
          }
          object.attachEvent('onclick',function(){
                  handler.call(object,arguments);
          });
  </script>
 ```
 2. 使用事件源代替this关键字，以下代码仅适用于IE6 IE7 IE8，这种方式完全忽略this关键字，但写起来稍显麻烦。
 ```
 <a id="xc">点击</a>
  <script type="text/javascript">
          function handler(e){
                  e=e||window.event;
                  var _this=e.srcElement||e.target;
                  alert(_this.innerHTML);
          }
          var object=document.getElementById('xc');
          object.attachEvent('onclick',handler);
  </script>
 ```
 3. 自己写一个函数完全代替attachEvent/detachEvent，并且支持所有主流浏览器、解决IE6 IE7 IE8事件绑定导致的先绑定后执行问题。下面是完美兼容addEventListener/removeEventListener和attachEvent/detachEvent的函数，支持Google Chrome/Firefox/Safari/Opera/IE 6 7 8 9 10 11等所有主流浏览器，能够完美解决IE6 IE7 IE8 this指向错误，能够纠正IE6 IE7 IE8中事件先绑定后执行的错误。为了避免代码被编辑器修改，请下载附件测试。不要直接复制下面的代码。注意，本函数是全局函数，而不是DOM对象的成员方法。
 ```
 <a id="xc">点击</a>
<script type="text/javascript">
        /*
         * 添加事件处理程序
         * @param object object 要添加事件处理程序的元素
         * @param string type 事件名称，如click
         * @param function handler 事件处理程序，可以直接以匿名函数的形式给定，或者给一个已经定义的函数名。匿名函数方式给定的事件处理程序在IE6 IE7 IE8中可以移除，在标准浏览器中无法移除。
         * @param boolean remove 是否是移除的事件，本参数是为简化下面的removeEvent函数而写的，对添加事件处理程序不起任何作用
        */
        function addEvent(object,type,handler,remove){
                if(typeof object!='object'||typeof handler!='function') return;
                try{
                        object[remove?'removeEventListener':'addEventListener'](type,handler,false);
                }catch(e){
                        var xc='_'+type;
                        object[xc]=object[xc]||[];
                        if(remove){
                                var l=object[xc].length;
                                for(var i=0;i<l;i++){
                                        if(object[xc][i].toString()===handler.toString()) object[xc].splice(i,1);
                                }
                        }else{
                                var l=object[xc].length;
                                var exists=false;
                                for(var i=0;i<l;i++){                                                
                                        if(object[xc][i].toString()===handler.toString()) exists=true;
                                }
                                if(!exists) object[xc].push(handler);
                        }
                        object['on'+type]=function(){
                                var l=object[xc].length;
                                for(var i=0;i<l;i++){
                                        object[xc][i].apply(object,arguments);
                                }
                        }
                }
        }
        /*
         * 移除事件处理程序
        */
        function removeEvent(object,type,handler){
                addEvent(object,type,handler,true);
        }
</script>
<script type="text/javascript">
        function handler(){
                alert(this.innerHTML);
        }
        var object=document.getElementById('xc');
        addEvent(object,'click',handler);
        //如果要测试绑定事件，请先删除下面这一行。
        removeEvent(object,'click',handler);
</script>
 ```

# script标签实现跨域

Web页面上凡是拥有"src"这个属性的标签都拥有跨域的能力，比如<script>、<img>、<iframe>页面引用JS文件，实际上返回的是一大段可执行的JS代码，浏览器在接收到了这些代码的时候会做相应的解析。我们同样也可以利用这一点来满足我们跨域请求数据的需求。
首先在客户端注册一个回调函数， 然后把回调函数的名字传给服务器。此时，服务器先生成 json 数据。然后以 javascript 语法的方式，生成一个function，function 名字就是传递上来的参数回调函数名。最后将 json数据直接以入参的方式，放置到function中，这样就生成了一段js语法的文档，返回给客户端。 客户端浏览器，解析script标签，并执行返回的javascript文档，此时数据作为参数，传入到了客户端预先定义好的 callback 函数里。(动态执行回调函数)
