---
tags: Vue.js
---

# Vue Composition API

## 你需要先知道的事

Composition API 是 vue3 開發出來的另外一種撰寫方法。

其好處是: 
  1. `setup()` 方法中沒有 `this`
  2. `setup()` 方法中可以只 `return` 實例需要用到的屬性
  3. 沒有命名衝突問題
  4. 其編譯後的程式碼更容易壓縮

另外建議使用 Composition API 都用 ESM 方式引入不要用 CDN 整包引入(ESM 跟 CDN 一次只選用一種，不要兩種都用)

在 `Composition API` 中 `data` `methods` `mounted` 都會捨棄，改寫成一個 `setup(){ return {}; }`

## 撰寫方式

創建一個 `html` 檔 並在 `body` 標籤中創建 `div#app` 在 `body` 最下面寫 `script` 代碼

首先使用 `ESM` 方式引入 `vue` 中的 `createApp`

接下來使用 `const` 聲明 `app` 為一個物件

物件中放入一個 `setup` 的函數 在裡面撰寫程式碼

最後通過 `Vue.createApp(app).mount('#app')` 創建整個 `vue` 實例

整個示例主要的 script 代碼如下：

```javascript=
import { createApp } from "https://cdnjs.cloudflare.com/ajax/libs/vue/3.0.11/vue.esm-browser.js";

const app = createApp({
  setup() {
    const text = ref('Hello World'); // 這裡等同於 data

    const changeText = () => { // 這裡等同於 methods
      text.value = 'Hello Vue';
    }

    return { // 宣告完後要用 return 才會渲染
      text,
      changeText
    }
  }
})

app.mount('#app');
```

## `ref` vs `reactive`

- ref 可用於變數物件或陣列等，總之就是不限型別，其內部還是調用 reactive
- reactive 只能用於物件，建議把有關聯的數據放在同一個對象裏提高代碼的可讀性（不熟的話先不要用，reactive 很多雷點）
- reactive 如果被解構賦值的話，裏面的東西會失去雙向綁定的特性，此時可以通過 toRefs 把物件裏面的每個屬性轉成 ref 來延續使用
- ref 定義的東西一定要用 .value 才可監控與獲取哦（多了 .value 就不會覆蓋到原始實體，相對來說會比較穩定）
- 了解更多請參考以下關鍵字：Proxy、getter、setter、defineproperty
- ref 用於物件時 在存取屬性也要加上 `.value` (EX: `person.value.name` .value 要放在物件的後面&屬性前面 小寫別寫錯喔)
- 建議初學者都是一路 const + ref 把 data 宣告到底（畢竟有雷別踩R~）
    - 建議使用 const 是要避免覆蓋，因為常數不能重新賦值（會報錯）
## `watch` vs `computed`

`vue` 中的 `watch` `computed` 可以單獨被拿出來使用

提取方法同上面的 `ref` 相同 直接通過解構方式即可(就是 `import { watch, computed } from "vue CDN URL";` 這樣)

在 `setup` 中要使用 `watch` `computed` 不需要像 `ref` `reactive` 一樣通過 `const` 定義

只需要直接寫 `watch()` `computed()` 即可，不過跟原本在 `vue` 實例中的寫法也稍有不同

首先 `watch` 的參數1 為 要監控的資料 `dataName`

  * 當要監控的資料為陣列或物件中的單一值 為了減輕效能 可以不需要把整個陣列或物件傳入 改成使用箭頭函式 `return` 該單一值
    - 這裏 `return` 可以讓它繼續被監控 
  * 需注意 若沒有使用 `return` 則該值就只是一個字符串 跟 `vue` 不會有任何綁定關係

參數2 為一個 `callback`，在 `callback` 中則一樣有 `newV` 與 `oldV`

這邊我們定義一個要監控的資料並把其傳入 `watch` 的參數1

再通過 `v-on` 更改其值 就會執行 `watch` 中的回調函數了

代碼如下：

```javascript=
import { createApp, ref, watch } from "https://cdnjs.cloudflare.com/ajax/libs/vue/3.0.11/vue.esm-browser.js";

const app = createApp({
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

    return { isShow, changeData }
  }
})

app.mount('#app')
```

`computed` 寫法就跟原本差不多 只需著要若要計算的值是用 ref 定義的 需要加上 `.value`

```javascript=
import { createApp, ref, watch, computed } from "https://cdnjs.cloudflare.com/ajax/libs/vue/3.0.11/vue.esm-browser.js";

const app = createApp({
  setup(){
    const num = ref(1)

    const dblNum = computed(() => {
      return num.value * 2
    })

    return { num, dblNum }
  }
})

app.mount('#app')
```

## mounted

mounted 在 Composition API 中要改寫成 onMounted 同一個 setup 中可以一次擁有多個 onMounted

比如以邏輯的思路把每個資料區域由上至下獨立分開設置

原本的寫法是必須把資料統一寫在 data 或 methods 或 mounted

Composition API 可以變成你想要哪些東西是有關聯性就寫在一起
不需要再強制性把東西都放在 data 或 methods 或 mounted

比如你要操作 text 相關的從 ref ，到 methods 的函數，到他的初始化都可以寫在最上方 接著下方就統一寫另一個假設是 num 相關的操作

在閱讀程式碼時可讀性會比較高 這也是其好處之一 所以可以多次重複調用 onMounted

