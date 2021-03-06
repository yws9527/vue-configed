                                          vue-resource插件使用
<hr/>
http://www.cnblogs.com/axl234/p/5899137.html<br/><hr/>

本文的主要内容如下：<br/><br/>
介绍vue-resource的特点<br/>
介绍vue-resource的基本使用方法<br/>
基于this.$http的增删查改示例<br/>
基于this.$resource的增删查改示例<br/>
基于inteceptor实现请求等待时的loading画面<br/>
基于inteceptor实现请求错误时的提示画面<br/>
本文11个示例的源码已放到GitHub，如果您觉得本篇内容不错，请点个赞，或在GitHub上加个星星！<br/><br/>
GitHub Source<br/>
本文的所有示例如下：<br/><br/>
http get示例<br/>
http jsonp示例<br/>
http post示例<br/>
http put示例<br/>
http delete示例<br/>
resource get示例<br/>
resource save示例(HTTP POST)<br/>
resource update示例(HTTP PUT)<br/>
resource remove示例(HTTP DELETE)<br/>
inteceptor示例1——ajax请求的loading界面<br/>
inteceptor实例2——请求失败时的提示对话框<br/>
各位在阅读这篇文章的内容时，可以先尝试该列表的最后两个示例，这两个示例综合使用了this.$http和inteceptor。<br/><br/>
vue-resource特点<br/>
vue-resource插件具有以下特点：<br/><br/>
1. 体积小<br/><br/>
vue-resource非常小巧，在压缩以后只有大约12KB，服务端启用gzip压缩后只有4.5KB大小，这远比jQuery的体积要小得多。<br/><br/>
2. 支持主流的浏览器<br/><br/>
和Vue.js一样，vue-resource除了不支持IE 9以下的浏览器，其他主流的浏览器都支持。<br/><br/>
3. 支持Promise API和URI Templates<br/><br/>
Promise是ES6的特性，Promise的中文含义为“先知”，Promise对象用于异步计算。<br/>
URI Templates表示URI模板，有些类似于ASP.NET MVC的路由模板。<br/><br/>
4. 支持拦截器 <br/><br/>
拦截器是全局的，拦截器可以在请求发送前和发送请求后做一些处理。<br/>
拦截器在一些场景下会非常有用，比如请求发送前在headers中设置access_token，或者在请求失败时，提供共通的处理方式。<br/><br/>
vue-resource使用<br/>
引入vue-resource<br/>
<script src="js/vue.js"></script><br/>
<script src="js/vue-resource.js"></script><br/>
基本语法<br/>
引入vue-resource后，可以基于全局的Vue对象使用http，也可以基于某个Vue实例使用http。<br/><br/>
// 基于全局Vue对象使用http<br/>
Vue.http.get('/someUrl', [options]).then(successCallback, errorCallback);<br/>
Vue.http.post('/someUrl', [body], [options]).then(successCallback, errorCallback);<br/><br/>
// 在一个Vue实例内使用$http<br/>
this.$http.get('/someUrl', [options]).then(successCallback, errorCallback);<br/>
this.$http.post('/someUrl', [body], [options]).then(successCallback, errorCallback);<br/>
在发送请求后，使用then方法来处理响应结果，then方法有两个参数，第一个参数是响应成功时的回调函数，第二个参数是响应失败时的回调函数。<br/><br/>
then方法的回调函数也有两种写法，第一种是传统的函数写法，第二种是更为简洁的ES 6的Lambda写法：<br/><br/>
// 传统写法<br/>
this.$http.get('/someUrl', [options]).then(function(response){<br/>
    // 响应成功回调
}, function(response){<br/>
    // 响应错误回调
});<br/><br/>
// Lambda写法<br/>
this.$http.get('/someUrl', [options]).then((response) => {<br/>
    // 响应成功回调
}, (response) => {<br/>
    // 响应错误回调
});
<br/>
PS：做过.NET开发的人想必对Lambda写法有一种熟悉的感觉。<br/><br/>
支持的HTTP方法<br/>
vue-resource的请求API是按照REST风格设计的，它提供了7种请求API：<br/>
<br/>
get(url, [options])<br/>
head(url, [options])<br/>
delete(url, [options])<br/>
jsonp(url, [options])<br/>
post(url, [body], [options])<br/>
put(url, [body], [options])<br/>
patch(url, [body], [options])<br/>
除了jsonp以外，另外6种的API名称是标准的HTTP方法。当服务端使用REST API时，客户端的编码风格和服务端的编码风格近乎一致，这可以减少前端和后端开发人员的沟通成本。<br/><br/>
客户端请求方法	服务端处理方法<br/>
this.$http.get(...)	Getxxx <br/>
this.$http.post(...)	Postxxx <br/>
this.$http.put(...)	Putxxx<br/>
this.$http.delete(...)	Deletexxx<br/>
options对象<br/>
发送请求时的options选项对象包含以下属性：<br/>
<br/>
参数	类型	描述<br/>
url	string	请求的URL<br/>
method	string	请求的HTTP方法，例如：'GET', 'POST'或其他HTTP方法<br/>
body	Object, FormDatastring	request body<br/>
params	Object	请求的URL参数对象<br/>
headers	Object	request header<br/>
timeout	number	单位为毫秒的请求超时时间 (0 表示无超时时间)<br/>
before	function(request)	请求发送前的处理函数，类似于jQuery的beforeSend函数<br/>
progress	function(event)	ProgressEvent回调处理函数<br/>
credientials	boolean	表示跨域请求时是否需要使用凭证<br/>
emulateHTTP	boolean	发送PUT, PATCH, DELETE请求时以HTTP POST的方式发送，并设置请求头的X-HTTP-Method-Override<br/>
emulateJSON	boolean	将request body以application/x-www-form-urlencoded content type发送<br/>
emulateHTTP的作用<br/><br/>
如果Web服务器无法处理PUT, PATCH和DELETE这种REST风格的请求，你可以启用enulateHTTP现象。启用该选项后，请求会以普通的POST方法发出，并且HTTP头信息的X-HTTP-Method-Override属性会设置为实际的HTTP方法。<br/><br/>
Vue.http.options.emulateHTTP = true;<br/>
emulateJSON的作用<br/><br/>
如果Web服务器无法处理编码为application/json的请求，你可以启用emulateJSON选项。启用该选项后，请求会以application/x-www-form-urlencoded作为MIME type，就像普通的HTML表单一样。<br/><br/>
Vue.http.options.emulateJSON = true;<br/>
response对象<br/>
response对象包含以下属性：<br/><br/>
方法	类型	描述<br/>
text()	string	以string形式返回response body<br/>
json()	Object	以JSON对象形式返回response body<br/>
blob()	Blob	以二进制形式返回response body<br/>
属性	类型	描述<br/>
ok	boolean	响应的HTTP状态码在200~299之间时，该属性为true<br/>
status	number	响应的HTTP状态码<br/>
statusText	string	响应的状态文本<br/>
headers	Object	响应头<br/>
注意：本文的vue-resource版本为v0.9.3，如果你使用的是v0.9.0以前的版本，response对象是没有json(), blob(), text()这些方法的。<br/><br/>
提示：以下示例仍然沿用上一篇的组件和WebAPI，组件的代码和页面HTML代码我就不再贴出来了。<br/><br/>
GET请求<br/>
var demo = new Vue({<br/>
    el: '#app',
    data: {<br/>
        gridColumns: ['customerId', 'companyName', 'contactName', 'phone'],
        gridData: [],
        apiUrl: 'http://211.149.193.19:8080/api/customers'
    },<br/>
    ready: function() {<br/>
        this.getCustomers()
    },<br/>
    methods: {<br/>
        getCustomers: function() {
            this.$http.get(this.apiUrl)
                .then((response) => {
                    this.$set('gridData', response.data)
                })<br/>
                .catch(function(response) {<br/>
                    console.log(response)
                })<br/>
        }<br/>
    }<br/>
})<br/>
<br/>
这段程序的then方法只提供了successCallback，而省略了errorCallback。<br/>
catch方法用于捕捉程序的异常，catch方法和errorCallback是不同的，errorCallback只在响应失败时调用，而catch则是在整个请求到响应过程中，只要程序出错了就会被调用。<br/><br/>
在then方法的回调函数内，你也可以直接使用this，this仍然是指向Vue实例的：<br/><br/>
getCustomers: function() {<br/>
    this.$http.get(this.apiUrl)
        .then((response) => {
            this.$set('gridData', response.data)
        })<br/>
        .catch(function(response) {
            console.log(response)
        })<br/>
}<br/>
<br/>
为了减少作用域链的搜索，建议使用一个局部变量来接收this。<br/><br/>
View Demo<br/>
<br/>
JSONP请求<br/>
getCustomers: function() {<br/>
    this.$http.jsonp(this.apiUrl).then(function(response){
        this.$set('gridData', response.data)
    })<br/>
}<br/>
POST请求<br/>
var demo = new Vue({<br/>
    el: '#app',
    data: {<br/>
        show: false,
        gridColumns: [{
            name: 'customerId',
            isKey: true
        }, {
            name: 'companyName'
        }, {
            name: 'contactName'
        }, {
            name: 'phone'
        }],<br/>
        gridData: [],<br/>
        apiUrl: 'http://211.149.193.19:8080/api/customers',<br/>
        item: {}<br/>
    },<br/>
    ready: function() {<br/>
        this.getCustomers()
    },<br/>
    methods: {<br/>
        closeDialog: function() {
            this.show = false
        },<br/>
        getCustomers: function() {<br/>
            var vm = this
            vm.$http.get(vm.apiUrl)
                .then((response) => {
                    vm.$set('gridData', response.data)
                })<br/>
        },<br/>
        createCustomer: function() {<br/>
            var vm = this
            vm.$http.post(vm.apiUrl, vm.item)
                .then((response) => {
                    vm.$set('item', {})
                    vm.getCustomers()
                })<br/>
            this.show = false<br/>
        }<br/>
    }<br/>
})<br/>
29<br/>
PUT请求<br/>
updateCustomer: function() {<br/>
    var vm = this
    vm.$http.put(this.apiUrl + '/' + vm.item.customerId, vm.item)<br/>
        .then((response) => {
            vm.getCustomers()
        })<br/>
}<br/>
30<br/>
Delete请求<br/>
deleteCustomer: function(customer){<br/>
    var vm = this
    vm.$http.delete(this.apiUrl + '/' + customer.customerId)<br/>
        .then((response) => {
            vm.getCustomers()
        })<br/>
}<br/>
31<br/>
使用resource服务<br/>
vue-resource提供了另外一种方式访问HTTP——resource服务，resource服务包含以下几种默认的action：<br/>
<br/>
get: {method: 'GET'},<br/>
save: {method: 'POST'},<br/>
query: {method: 'GET'},<br/>
update: {method: 'PUT'},<br/>
remove: {method: 'DELETE'},<br/>
delete: {method: 'DELETE'}<br/>
resource对象也有两种访问方式：<br/><br/>
全局访问：Vue.resource<br/>
实例访问：this.$resource<br/>
resource可以结合URI Template一起使用，以下示例的apiUrl都设置为{/id}了：<br/><br/>
apiUrl: 'http://211.149.193.19:8080/api/customers{/id}'<br/>
GET请求<br/>
使用get方法发送GET请求，下面这个请求没有指定{/id}。<br/><br/>
getCustomers: function() {<br/>
    var resource = this.$resource(this.apiUrl)
        vm = this 
        resource.get()<br/>
        .then((response) => {<br/>
            vm.$set('gridData', response.data)
        })<br/>
        .catch(function(response) {<br/>
            console.log(response)
        })<br/>
}<br/>

POST请求<br/>
使用save方法发送POST请求，下面这个请求没有指定{/id}。<br/>
<br/>
createCustomer: function() {<br/>
    var resource = this.$resource(this.apiUrl)
        vm = this
    resource.save(vm.apiUrl, vm.item)<br/>
        .then((response) => {
            vm.$set('item', {})
            vm.getCustomers()<br/>
        })<br/>
    this.show = false<br/>
}<br/><br/>
PUT请求<br/>
使用update方法发送PUT请求，下面这个请求指定了{/id}。<br/>
<br/>
updateCustomer: function() {<br/>
    var resource = this.$resource(this.apiUrl)
        vm = this
        
    resource.update({ id: vm.item.customerId}, vm.item)<br/>
        .then((response) => {
            vm.getCustomers()
        })<br/>
}<br/>
{/id}相当于一个占位符，当传入实际的参数时该占位符会被替换。<br/>
例如，{ id: vm.item.customerId}中的vm.item.customerId为12，那么发送的请求URL为：<br/>
<br/>
http://211.149.193.19:8080/api/customers/12<br/><br/>
DELETE请求<br/>
使用remove或delete方法发送DELETE请求，下面这个请求指定了{/id}。<br/>
<br/>
deleteCustomer: function(customer){<br/>
    var resource = this.$resource(this.apiUrl)
        vm = this
        
    resource.remove({ id: customer.customerId})<br/>
        .then((response) => {
            vm.getCustomers()
        })<br/>
}<br/><br/>
使用inteceptor<br/>
拦截器可以在请求发送前和发送请求后做一些处理。<br/><br/>
基本用法<br/>
Vue.http.interceptors.push((request, next) => {<br/>
        // ...
        // 请求发送前的处理逻辑
        <br/>
        // ...
        <br/>
    next((response) => {<br/>
        // ...
        <br/>
        // 请求发送后的处理逻辑
        <br/>
        // ...
        <br/>
        // 根据请求的状态，response参数会返回给successCallback或errorCallback
        <br/>
        return response
        <br/>
    })<br/>
})<br/>
在response返回给successCallback或errorCallback之前，你可以修改response中的内容，或做一些处理。<br/>
例如，响应的状态码如果是404，你可以显示友好的404界面。<br/><br/>
如果不想使用Lambda函数写法，可以用平民写法：
<br/><br/><br/>
Vue.http.interceptors.push(function(request, next) {<br/>
    // ...
    <br/>// 请求发送前的处理逻辑
    <br/>// ...
    next(function(response) {<br/>
        // ...
        <br/>// 请求发送后的处理逻辑
        <br/>// ...
        <br/>// 根据请求的状态，response参数会返回给successCallback或errorCallback
        <br/>return response
    })<br/>
})<br/>
示例1<br/>
之前的CURD示例有一处用户体验不太好，用户在使用一些功能的时候如果网络较慢，画面又没有给出反馈，用户是不知道他的操作是成功还是失败的，他也不知道是否该继续等待。<br/><br/>
通过inteceptor，我们可以为所有的请求处理加一个loading：请求发送前显示loading，接收响应后隐藏loading。<br/><br/>
具体步骤如下：<br/><br/>
1.添加一个loading组件<br/><br/>
<template id="loading-template"><br/>
    <div class="loading-overlay"><br/>
        <div class="sk-three-bounce"><br/>
            <div class="sk-child sk-bounce1"></div><br/>
            <div class="sk-child sk-bounce2"></div><br/>
            <div class="sk-child sk-bounce3"></div><br/>
        </div><br/>
    </div><br/>
</template><br/>
2.将loading组件作为另外一个Vue实例的子组件<br/><br/>
var help = new Vue({<br/>
    el: '#help',
    data: {<br/>
        showLoading: false
    },<br/>
    components: {<br/>
        'loading': {
            template: '#loading-template',
        }<br/>
    }<br/>
})<br/>
3.将该Vue实例挂载到某个HTML元素<br/><br/>
<div id="help"><br/>
    <loading v-show="showLoading"></loading><br/>
</div><br/>
4.添加inteceptor<br/><br/>
Vue.http.interceptors.push((request, next) => {<br/>
    loading.show = true
    next((response) => {<br/>
        loading.show = false
        return response
    });<br/>
});<br/>
27<br/>

示例2<br/>
当用户在画面上停留时间太久时，画面数据可能已经不是最新的了，这时如果用户删除或修改某一条数据，如果这条数据已经被其他用户删除了，服务器会反馈一个404的错误，但由于我们的put和delete请求没有处理errorCallback，所以用户是不知道他的操作是成功还是失败了。<br/><br/>
你问我为什么不在每个请求里面处理errorCallback，这是因为我比较懒。这个问题，同样也可以通过inteceptor解决。<br/><br/>
1. 继续沿用上面的loading组件，在#help元素下加一个对话框<br/><br/>
<div id="help"><br/>
    <loading v-show="showLoading" ></loading><br/>
    <modal-dialog :show="showDialog"><br/>
        <header class="dialog-header" slot="header"><br/>
            <h1 class="dialog-title">Server Error</h1><br/>
        </header><br/>
        <div class="dialog-body" slot="body"><br/>
            <p class="error">Oops,server has got some errors, error code: {{errorCode}}.</p><br/>
        </div><br/>
    </modal-dialog><br/>
</div><br/>
2.给help实例的data选项添加两个属性<br/><br/>
var help = new Vue({<br/>
        el: '#help',
        data: {
            showLoading: false,
            showDialog: false,
            errorCode: ''
        },<br/>
        components: {<br/>
            'loading': {
                template: '#loading-template',
            }<br/>
        }<br/>
    })<br/>
3.修改inteceptor<br/><br/>
Vue.http.interceptors.push((request, next) => {<br/>
    help.showLoading = true
    next((response) => {<br/>
        if(!response.ok){
            help.errorCode = response.status
            help.showDialog = true
        }<br/>
        <br/>help.showLoading = false
        <br/>return response
    });<br/>
});<br/>
28<br/><br/>
总结<br/>
vue-resource是一个非常轻量的用于处理HTTP请求的插件，它提供了两种方式来处理HTTP请求：<br/><br/>
使用Vue.http或this.$http<br/>
使用Vue.resource或this.$resource<br/>
这两种方式本质上没有什么区别，阅读vue-resource的源码，你可以发现第2种方式是基于第1种方式实现的。<br/><br/>
inteceptor可以在请求前和请求后附加一些行为，这意味着除了请求处理的过程，请求的其他环节都可以由我们来控制。<br/><br/>
