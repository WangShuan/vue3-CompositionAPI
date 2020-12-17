# Composition API

Composition API 是 vue3 開發出來的另外一種撰寫方法。

其好處是: 

  1. `setup()` 方法中沒有 `this`
  2. `setup()` 方法中可以只 `return` 實例需要用到的屬性
  3. 沒有命名衝突問題
  4. 其編譯後的程式碼更容易壓縮

---

## 1. 撰寫方式

創建一個 `html` 檔 並使用 `CDN` 引入 `vue3`

在 `body` 標籤中創建 `div#app` 

在最下面寫 `script` 代碼

首先使用 `ES6` 解構方式引入 `vue` 中的 `ref`

接下來使用 `const` 聲明 `app` 為一個物件

物件中放入一個 `setup` 的函數 在裡面撰寫 `vue`

最後通過 `Vue.createApp(app).mount('#app')` 創建整個 `vue` 實例

整個示例主要的 script 代碼如下：

```js

const { ref } = Vue
  
const app = {
  setup() {
    const text = ref('Hello World')

    const changeText = () => {
      text.value = 'Hello Vue';
    }

    return {
      text,
      changeText
    }
  }
}

Vue.createApp(app).mount('#app')

```

### 1-1. `ref` vs `reactive`

* `ref` 用來做 `vue` 資料傳遞 任何形式的 `data` 都可以通過 `ref` 傳遞

  - 要注意的是 `ref` 的資料需通過 `dataName.value` 才可更改其內容

  - 另外當 `ref` 資料為陣列或物件時 是無法做深層的監聽 所以使用 `reactive` 傳遞陣列與物件會更合適

* `reactive` 只能接受 _陣列或物件_ 的資料傳遞

  - 可以直接使用 `dataName.xxx` 來更改資料內容

  - 與 `ref` 不同 不需要通過 `.value` 修改值

### 1-2. `watch`

`vue` 中的 `watch` 可以單獨被拿出來使用

提取方法同上面的 `ref` 相同 直接通過解構方式即可

在 `setup` 中要使用 `watch` 函數不需要像 `ref` `reactive` 一樣通過 `const` 定義

只需要直接寫 `watch()` 即可

不過跟原本在 `vue` 實例中的寫法也稍有不同

首先 `watch` 的參數1 為 要監控的資料 `dataName`

  * 當要監控的資料為陣列或物件中的單一值 為了減輕效能 
  可以不需要把整個陣列或物件傳入 改成使用箭頭函式 `return` 該單一值
  
    - 這裏 `return` 可以讓它繼續被監控 
    若沒有使用 `return` 則該值就只是一個字符串 跟 `vue` 不會有任何綁定關係

參數2 為一個 `callback` 在 `callback` 中則一樣有 `newV` 與 `oldV`

我們定義一個要監控的資料並把其傳入 `watch` 參數1

再通過 `v-on` 更改其值 就會執行 `watch` 中的回調函數了

代碼如下：

```js

const { ref, watch } = Vue

const app = {
  setup(){
    const isShow = ref(false)
    const formData = reactive({ name:'', age:10 })

    const changeData = () => {
      isShow.value = !isShow.value
      formData.age = 19
    }

    watch(isShow, (newV,oldV) => {
      console.log(newV, oldV)
    })

    // 監控物件中的單一值
    watch(() => formData.age, ( newV, oldV ) => {
      console.log(newV, oldV)
    })

    return { isShow, changeShow }
  }
}

Vue.createApp(app).mount('#app')

```