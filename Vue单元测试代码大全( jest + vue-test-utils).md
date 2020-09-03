> 人(ce)生(shi)如戏，全靠演技(mock)。

### 前言
对于vue单元测试。如果你翻遍文档，阅教程无数，还是感觉差了那么点意思。
那么我斗胆断言：你与神功大成只是差了一个例子的距离。  

这份“vue单元测试最佳实践”就是专门为你准备的礼物。
一定先收藏上，不难预见，当你真正需要并去看的时候，会发自内心的来上一句：不虚此藏。

技术栈：jest、vue-test-utils。  
共四个部分：运行时、Mock、Stub、Configuring和CLI。  
### 运行时
在跑测试用例时，大家的第一个绊脚石肯定是各种undifned报错。 

解决这些报错的血泪史还历历在目，现在总结来看，大都是缺少运行时变量抑或异步造成的。  
这里咱们只说运行时，基本就这两类：
#### 1. 缺少window等环境变量
一般通过引入global-jsdom解决，这也是官方推荐的。当然我们也可以自己在测试代码中直接声明定义。  

比如我们在业务代码中使用了sessionStorage。
```js
// procudtpay.vue
<script>
const sessionParams = window.sessionStorage.getItem('sessionParams')
export default {
  data () { }
}
</script>
```
然后在测试代码中直接重定义，这样在运行时，实际取到的值就是我们在这里定义的。
```js
// procudtpay.spec.js
window.sessionStorage = {
  getItem: () => {
    return { name:'name', type:'type' }
  }
}
import procudtpay from '../views/procudtpay.vue'
```
这里关于执行顺序做一点额外说明：  

示例中sessionParams的赋值是在import引入.vue模块就执行了的，所以对sessionStorage的定义赋值需要在引入之前。  

如果你的sessionStorage取值是在vue实例化后，比如created中，那么则没有该问题。
#### 2. 缺少在main.js中定义/注册的全局属性和方法
这些就需要在测试代码中引入同款，以及通过mount的配置项mocks和stubs，分别对其进行mock或者存根了。
```js
// main.js
import Vue from 'vue'
import Mint from 'mint-ui'
import '../filter'
import axios from 'axios'
Vue.use(Mint)
Vue.prototype.$post = (url, params) => {
  return axios.post(url, params).then(res => res.data)
}
Vue.filter('filterxxx', function (value) {
  // bala bala ba…
})

// xxx.spec.js
import Vue from 'vue'
import '../../filter/filter'   // 引入注册同款过滤器
Vue.filter('filterxxx', function (value) {
  // bala bala ba…
})
import { $post } from './http.js' 
it('快照测试', () => {
    const wrapper = shallowMount(ProductPay, {
      mocks: {
        $post  // 用自己定义的mock数据取代真实http请求
      },
      stubs:['mt-header'] // 存根组件
    })
    // ...
})
```
通常其他测试文件也会依赖这些全局变量，我们可以通过[配置jest的setupFiles](https://jestjs.io/docs/zh-Hans/configuration#setupfiles-array)实现复用。
### Mock
我翻开代码一看，这代码没有注释，歪歪斜斜的每一行都写着‘断言正确’四个字。我横竖睡不着，仔细看了半夜，才从字缝里看出字来，满屏都写着两个字：‘造假’！

正应了那一句：人(ce)生(shi)如戏，全靠演技(mock)。总之，mock老重要了。
#### 1. mock 简单函数
我们从最简单的mock一个函数开始。  

比如我们现在想要测试：当用户购买成功，期望页面能跳转到结果页。
```js
// productpay.vue
<script>
export default {
    ...
    methods:{
		commmit () {
		  this.$post('xxx', params).then(data => {
            this.$router.push(`/payresult`)
        })
	   }
	}
}
</script>
```
那么，我们可以通过mock掉$router的push方法，然后断言它有被调用且参数正确，达成测试目的。
```js
// productpay.spec.js
it('当用户购买成功后，页面应该跳转至结果页', async () => {
    const mockFunc = jest.fn()
    const wrapper = shallowMount(ProductPay, {
      mocks: {
        $post,
        $router: {
          push: mockFunc
        }
      }
    })
    
    wrapper.vm.commmit() // 提交购买
    
    expect(mockFunc).toHaveBeenCalledWith('/payresult')
})
```
#### 2. mock Http请求，指定返回结果
http请求和上面例子中的$router的区别是，它需要返回值。jest有多种方式指定返回值，这里用的是mockImplementation。
```js
// test/**.spec.js
it('当用户xxxx，应该xxxx', async () => {
	const respSuccess = { data: [...], code:0 }
	const respError = { data: [...], code:888 }
	// 定义mock函数
	const mockPost = jest.fn() 
	const wrapper = shallowMount(index, { 
	   mocks: {
        $post:mockPost // 应用该mock函数
        }
   })
   // 指定异步返回数据
   mockPost.mockImplementation(() => Promise.resolve(respError))
   // 可以对调用情况进行断言
   expect(mockPost).toHaveBeenCalled() 
  
   mockPost.mockImplementation(() => Promise.resolve(respSuccess))
   //也可以等待异步结束，对结果进行断言
   await flushPromises()
   expect(wrapper.vm.list).toEqual(respSuccess.data)
})
```
实际上我们项目中调用的接口会很多，且不乏返回大量数据的情况。如果这些都定义在测试代码里就会很臃肿。这时候，我们可以对该功能做个简单的模块化。
```js
// 常见的业务代码
// main.js中把axios挂载到了vue实例
Vue.prototype.$post = (url, params) => {
  return axios.post(url, params).then(res => res.data)
}
// Index.vue中的请求
getProductList () {
    this.$post('/ProductListQry', {}).then(data => {
        this.ProductList = data.List
    })
}
```
```js
// 1. 在单独js中存放模拟数据 data/ProductListQry.js
export default {
	data:[{ id:1,name:'name',...},...],
	code:0
}

// 2. 定义post方法，并做个数据匹配 test/http.js
import ProductListQry from '@/data/ProductListQry.js'
const mockData = {
  ProductListQry,
  ... //可以用同样的方式引入更多mock数据
}
const $post = (url = '') => {
  return new Promise((resolve, reject) => {
    const jsName = String(url).split('/')[1]
    resolve(mockData[jsName])
  })
}
export { $post }

// 3. 引入并使用 test/index.spec.js
import Index from '@/views/Index.vue'
import { $post } from './http.js'
it('...',()=>{
    const wrapper = shallowMount(Index, {
      mocks: {
        $post
      }
    })
    wrapper.vm.getProductList() //触发请求
    await flushPromises() //等待异步请求结束
    //可以看到wrapper中就有了我们指定的模拟数据
    console.log(wrapper.vm.ProductList) 
})
```
同理，如果要测试请求失败的情形，可以再定义一个返回错误数据的方法，比如就叫$postError。
```js
// test/**.spec.js
import { $postError } from './http.js'
it('...',()=>{
    const wrapper = shallowMount(Index, {
        mocks: {
            $post:$postError
        }
    })
    
    wrapper.vm.getProductList() //触发请求
    await flushPromises() //等待异步请求结束
    
    // 我们就可以就获取到错误数据的场景进行测试了
    console.log(wrapper.vm.ProductList) 
})
```
#### 3. mock 整个模块
当业务代码中直接使用了引入的组件/方法时，我们对其测试可能就需要mock整个模块。
下面是一个用弹窗做表单验证的场景：
```js
// productpay.vue
<script>
import { MessageBox } from '../Component'
export default {
    methods:{
        makeSurebuy () {
            let payAmount = delcommafy(this.payAmount)
                if (!payAmount) {
                    MessageBox({
                    message: '请先输入购买金额'
                })
                return
            }
            if (payAmount < this.resData.BaseAmt) {
                MessageBox({
                    message: '购买金额不能小于起存金额'
                })
                return
            }
            if (payAmount > this.Balance) {
                MessageBox({
                    message: '购买金额不能大于可用余额'
                })
                return
            }
            // 校验通过，发起交易...
        }
    }
}
<script>
```
```js
//productpay.spce.js
import Component from '../Component'
jest.mock('../../../components/ZyComponent')

it('当用户点击购买按钮，如果输入非法金额，应该有相应的错误提示', async () => {
    wrapper.findAll('.btn-commit').at(0).trigger('click')
    expect(Component.MessageBox.mock.calls[0][0])
        .toEqual({ message: '请先输入购买金额' })
    
    wrapper.setData({payAmount: '100'})
    
    wrapper.findAll('.btn-commit').at(0).trigger('click')
    expect(Component.MessageBox.mock.calls[1][0])
        .toEqual({ message: '购买金额不能小于起存金额' })
    
    wrapper.setData({payAmount: '100000000000000000'})
    
    wrapper.findAll('.btn-commit').at(0).trigger('click')
    expect(Component.MessageBox.mock.calls[2][0])
        .toEqual({ message: '购买金额不能大于可用余额' })
})
```
我们通过jest.mock() mock整个模块，当该模块的方法被调用后它就会有一个mock属性，可以通过ZyComponent.ZyMessageBox.mock进行访问，其中ZyComponent.ZyMessageBox.mock.calls会返回被调用情况的数组，我们可以根据这个数据对函数被调用次数、入参情况进行断言测试。
### Stub存根组件
进行单元测试，理论上我们不用、也不应该在它的测试用例中测试子组件，不然就叫集成测试了。
vue-test-utils是通过配置stubs实现对组件mock的。
```js
const wrapper = shallowMount(index, {
    stubs: ['mt-header', 'mt-loadmore']
}
```
但是业务中难免会有调用子组件方法的时候，比如说mint-ui的loadmore。
```js
// procuctlist.vue
<script>
export default {
    ...
    methods:{
		getProductList () {
		  this.$post('xxx', params).then(data => {
		  ...
            this.ProductList = this.ProductList.concat(data.List)
            this.$refs.loadmore.onBottomLoaded()
        })
	   }
	}
}
</script>
```
这时候我们是可以改用mount方法使页面渲染子组件，这样通过$refs就能正常的获取到子组件实例。但更合适的做法应该是自定义存根组件的内部实现，以满足测试需求。
```js
// procuctlist.spec.js
it('当用户上拉产品列表，应该能看到的更多的产品', () => {
    const mockOnBottomLoaded = jest.fn()
    const mtLoadMore = {
      render: () => { },
      methods: {
        onBottomLoaded: mockOnBottomLoaded
      }
    }
    const mtHeader = {
      render: () => { }
    }
    const wrapper = shallowMount(Index, {
      stubs: { 'mt-loadmore': mtLoadMore, 'mt-header': mtHeader },
      mocks: {
        $post
      }
    })
    const currentPage = wrapper.vm.currentPage

    wrapper.vm.loadMoreProduction()

    expect(wrapper.vm.currentPage).toEqual(currentPage + 1)
    expect(mockOnBottomLoaded).toHaveBeenCalled()
})
```
最后提一嘴，存根组件后，业务代码中子组件还是会被引入的，只是没有被实例化和渲染。
### Configuring和CLI
#### 1. 统计代码覆盖率忽略某些文件
使用coveragePathIgnorePatterns配置即可，把这个列出来是应为我遇到两个项目相同配置，有一个死活不生效的问题。最后才从[官方文档](https://jestjs.io/docs/zh-Hans/troubleshooting#coveragepathignorepatterns-seems-to-not-have-any-effect)中得知是babel插件istanbul问题。目前还未解决，只是粗暴的在.balelrc中把istanbul去掉了。有真正解决方案的大佬，留言教下……跪谢。
```js
// jest.config.js
{
    coveragePathIgnorePatterns: ['<rootDir>/src/assets/']
}
```
#### 2. 通过t模式，可以仅执行指定的测试用例
当测试用例写的多了，每次执行跑一堆用例，效率很低，如果代码里有很多console, 那就更难受了，找个报错都能找半天。当时就想如果能仅测试当前用例就好了。
然后就找到了t模式，jest命令带--watch参数进入监听模式，然后输入t,再输入匹配规则即可。世界一下子就清净了，舒服……
```json
// package.json
{
    "scripts":{
        "tets":"jest --watch"
    }
}
```
#### 3. vue-awesome-swiper测试运行时报错
如果组件中引入了swiper，那么在执行测试用例时，vue-awesome-swiper中的js会报错。引用即报错，且是第三方代码。  
最后通过把swiper组件由局部注册改为全局注册得以解决。
### 感谢
最后列一下在学习过程中看到的对我帮助很大的文章和资料
* [Jest官方文档](https://jestjs.io/zh-Hans/)
* [vue-test-utils官方指南](https://vue-test-utils.vuejs.org/zh/)
* [一篇文章学会 Vue 项目单元测试](https://zhuanlan.zhihu.com/p/48758013)
* [Jest结合Vue-test-utils使用的初步实践](https://blog.csdn.net/duola8789/article/details/80434962)