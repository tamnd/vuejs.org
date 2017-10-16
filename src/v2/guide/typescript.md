---
title: Hỗ trợ Typescript
type: guide
order: 404
---

## Những chú ý về thay đổi quan trọng từ phiên bản 2.2.0+ cho người dùng Typescript + Webpack 2

Từ phiên bản Vue 2.2.0+, chúng tôi đã đóng gói các tập tin phân phối (dist files) dưới dạng ES module, và có thể dùng trực tiếp với Webpack 2. Nhưng sự thay đổi này dẫn đến [..] breaking changes với người dùng Typescript + Webpack 2, vì khi đó lệnh `import Vue = require('Vue')` bây giờ sẽ trả về một synthetic ES module object, thay vì chính object Vue.

Chúng tôi có kế hoạch chuyển tất cả các khai báo (declaration) sang export kiểu ES trong tương lai. Vì thế hãy sử dụng cấu hình được đề xuất dưới đây cho các ứng dụng tương lai của bạn.

## Khai báo trong các gói NPM (NPM package)

Một hệ thống kiểu tĩnh có thể giúp phát hiện được nhiều lỗi thời gian chạy (runtime error), đặc biệt khi ứng dụng trở lên lớn. Đó cũng là lý do tại sao Vue đi kèm với [khai báo kiểu]() cho [Typescript](https://typescriptlang.org/) - không chỉ core Vue, mà cả [vue-router](https://github.com/vuejs/vue-router/tree/dev/types) và [vuex](https://github.com/vuejs/vuex/tree/dev/types).  

Vì các gói được phát hành trên NPM](), và phiên bản mới nhất của Typescript hiểu dược các khai báo kiểu trong các gói NPM, nên khi cài đặt Vue bằng NPM, bạn không cần phải cài thêm bất kì thứ gì khác để có thể sử dụng Typescript với Vue.

## Cấu hình đề xuất
## Recommended Configuration

``` js
// tsconfig.json
{
  "compilerOptions": {
    // ... other options omitted
    "allowSyntheticDefaultImports": true,
    "lib": [
      "dom",
      "es5",
      "es2015.promise"
    ]
  }
}
```

Chú ý tuỳ chọn `allowSyntheticDefaultImports` cho phép sử dụng những điều sau:

``` js
import Vue from 'vue'
```

thay vì:

``` js
import Vue = require('vue')
```

Cú pháp module kiểu ES được khuyến cáo vì nó phù hợp với cách sử dụng ES, và trong tương lai, chúng tôi đang có kế hoạch chuyển toàn bộ khai báo (declaration) sang sử dụng export kiểu ES.

Ngoài ra, nếu bạn đang sử dụng Typescript với Webpack 2, bạn cũng nên dùng cấu hình như sau:

``` js
{
  "compilerOptions": {
    // ... other options omitted
    "module": "es2015",
    "moduleResolution": "node"
  }
}
```

Cấu hình nhưu này cho phép Typescript giữ nguyên các lệnh import ES module, khi đó Webpack 2 sẽ tận dụng được tính năng loại bỏ các module và đoạn code thừa trên các ES module.

Bạn có thể xem thêm tài liệu về [các tuỳ chọn của trình biên dịch Typescript](https://www.typescriptlang.org/docs/handbook/compiler-options.html) để biết thêm chi tiết. 

## Using Vue's Type Declarations

Vue's type definition exports many useful [type declarations](https://github.com/vuejs/vue/blob/dev/types/index.d.ts). For example, to annotate an exported component options object (e.g. in a `.vue` file):

``` ts
import Vue, { ComponentOptions } from 'vue'

export default {
  props: ['message'],
  template: '<span>{{ message }}</span>'
} as ComponentOptions<Vue>
```

## Class-Style Vue Components

Vue component options can easily be annotated with types:

``` ts
import Vue, { ComponentOptions }  from 'vue'

// Declare the component's type
interface MyComponent extends Vue {
  message: string
  onClick (): void
}

export default {
  template: '<button @click="onClick">Click!</button>',
  data: function () {
    return {
      message: 'Hello!'
    }
  },
  methods: {
    onClick: function () {
      // TypeScript knows that `this` is of type MyComponent
      // and that `this.message` will be a string
      window.alert(this.message)
    }
  }
// We need to explicitly annotate the exported options object
// with the MyComponent type
} as ComponentOptions<MyComponent>
```

Unfortunately, there are a few limitations here:

- __TypeScript can't infer all types from Vue's API.__ For example, it doesn't know that the `message` property returned in our `data` function will be added to the `MyComponent` instance. That means if we assigned a number or boolean value to `message`, linters and compilers wouldn't be able to raise an error, complaining that it should be a string.

- Because of the previous limitation, __annotating types like this can be verbose__. The only reason we have to manually declare `message` as a string is because TypeScript can't infer the type in this case.

Fortunately, [vue-class-component](https://github.com/vuejs/vue-class-component) can solve both of these problems. It's an official companion library that allows you to declare components as native JavaScript classes, with a `@Component` decorator. As an example, let's rewrite the above component:

``` ts
import Vue from 'vue'
import Component from 'vue-class-component'

// The @Component decorator indicates the class is a Vue component
@Component({
  // All component options are allowed in here
  template: '<button @click="onClick">Click!</button>'
})
export default class MyComponent extends Vue {
  // Initial data can be declared as instance properties
  message: string = 'Hello!'

  // Component methods can be declared as instance methods
  onClick (): void {
    window.alert(this.message)
  }
}
```

With this syntax alternative, our component definition is not only shorter, but TypeScript can also infer the types of `message` and `onClick` without explicit interface declarations. This strategy even allows you to handle types for computed properties, lifecycle hooks, and render functions. For full usage details, see [the vue-class-component docs](https://github.com/vuejs/vue-class-component#vue-class-component).

## Declaring Types of Vue Plugins

Plugins may add to Vue's global/instance properties and component options. In these cases, type declarations are needed to make plugins compile in TypeScript. Fortunately, there's a TypeScript feature to augment existing types called [module augmentation](https://www.typescriptlang.org/docs/handbook/declaration-merging.html#module-augmentation).

For example, to declare an instance property `$myProperty` with type `string`:

``` ts
// 1. Make sure to import 'vue' before declaring augmented types
import Vue from 'vue'

// 2. Specify a file with the types you want to augment
//    Vue has the constructor type in types/vue.d.ts
declare module 'vue/types/vue' {
  // 3. Declare augmentation for Vue
  interface Vue {
    $myProperty: string
  }
}
```

After including the above code as a declaration file (like `my-property.d.ts`) in your project, you can use `$myProperty` on a Vue instance.

```ts
var vm = new Vue()
console.log(vm.$myProperty) // This will be successfully compiled
```

You can also declare additional global properties and component options:

```ts
import Vue from 'vue'

declare module 'vue/types/vue' {
  // Global properties can be declared
  // by using `namespace` instead of `interface`
  namespace Vue {
    const $myGlobal: string
  }
}

// ComponentOptions is declared in types/options.d.ts
declare module 'vue/types/options' {
  interface ComponentOptions<V extends Vue> {
    myOption?: string
  }
}
```

The above declarations allow the following code to be compiled:

```ts
// Global property
console.log(Vue.$myGlobal)

// Additional component option
var vm = new Vue({
  myOption: 'Hello'
})
```
