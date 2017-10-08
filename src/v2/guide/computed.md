---
title: Thuộc tính computed và watcher
type: guide
order: 5
---

## Thuộc tính computed

Những biểu thức đơn giản được viết bên trong template sẽ rất tiện lợi. Tuy nhiên, những biểu thức phức tạp được viết theo cách đó sẽ khiến template cồng kềnh và khó bảo trì. Ví dụ như:

``` html
<div id="example">
  {{ message.split('').reverse().join('') }}
</div>
```

Ở trường hợp này, template không còn đơn giản và mang tính khai báo nữa (declarative). Bạn phải mất chút thời gian thì mới nhận ra được `message` được đảo ngược. Vấn đề sẽ trở nên tồi tệ nếu bạn sử dụng cách này lập đi lập lại trong template.

Đó là lí do tại sao bạn nên dùng **thuộc tính computed** cho những biểu thức phức tạp.

### Ví dụ căn bản

``` html
<div id="example">
  <p>Nội dung gốc: "{{ message }}"</p>
  <p>Nội dung được đảo ngược (computed): "{{ reversedMessage }}"</p>
</div>
```

``` js
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // một computed getter
    reversedMessage: function () {
      // `this` trỏ tới đối tượng vm
      return this.message.split('').reverse().join('')
    }
  }
})
```

Kết quả là:

{% raw %}
<div id="example" class="demo">
  <p>Nội dung gốc: "{{ message }}"</p>
  <p>Nội dung được đảo ngược (computed): "{{ reversedMessage }}"</p>
</div>
<script>
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    reversedMessage: function () {
      return this.message.split('').reverse().join('')
    }
  }
})
</script>
{% endraw %}

Ở đây chúng ta khai báo một thuộc tính computed là `reversedMessage`. Hàm được dùng như trên sẽ là một hàm getter cho thuộc tính `vm.reversedMessage`:

``` js
console.log(vm.reversedMessage) // => 'olleH'
vm.message = 'Goodbye'
console.log(vm.reversedMessage) // => 'eybdooG'
```

Bạn có thể mở console và thử chạy ví dụ đảo ngược chữ. Gíá trị của `vm.reversedMessage` luôn phụ thuộc vào giá trị của `vm.message`.

Bạn có thể ràng buộc dữ liệu (data-bind) cho những thuộc tính computed trong template một cách bình thường như những thuộc tính khác. Vue biết được `vm.reversedMessage` phụ thuộc vào `vm.message`, vì vậy Vue sẽ cập nhất bất kì ràng buộc (binding) nào phụ thuộc vào `vm.reversedMessage` khi `vm.message` thay đổi. Phần tốt nhất đó là chúng ta tạo ra được mối liên hệ giữa các dependency: các hàm getter của computed thì không bị side effect (nhiệm vụ của getter chỉ là xử lí và trả về giá trị cho thuộc tính computed), chính điều đó giúp dễ hiểu và dễ kiểm tra (test).

### Computed caching và phương thức

Bạn sẽ nhận thấy rằng chúng ta cũng có thể đạt được nội dung đảo ngược bằng cách sử dụng một phương thức:

``` html
<p>Nội dung đảo ngược: "{{ reverseMessage() }}"</p>
```

``` js
// trong component
methods: {
  reverseMessage: function () {
    return this.message.split('').reverse().join('')
  }
}
```

Thay vì sử dụng thuộc tính computed, chúng ta cũng có thể dùng một phương thức thay thế. Nếu xét về kết quả cuối cùng thì hai cách tiếp cận này thật sự là như sau. Tuy nhiên, sự khác biệt ở đây là **những thuộc tính computed được cache lại dựa vào những dependency (những giá trị mà computed phụ thuộc vào).** Một thuộc tính computed chỉ được tính toán lại khi những dependency của chúng thay đổi. Điều này có nghĩa như sau: nếu giá trị của `message` không thay đổi, thì những truy cập tới computed `reversedMessage` sẽ ngay lập tức trả về kết quả được tính toán trước đó mà không phải chạy lại hàm một lần nữa.

Điều này cũng có nghĩa giá trị của một thuộc tính computed sẽ không được cập nhật nếu dependency của nó không thay đổi. Ví dụ phía dưới, computed `now` sẽ không được cập nhật vì `Date.now` không phải là một reactive dependency:

``` js
computed: {
  now: function () {
    return Date.now()
  }
}
```

Về phần phương thức, chúng sẽ **luôn** được gọi khi có một sự kiện re-render diễn ra.

Tại sao chúng ta lại cần việc cache dữ liệu? Thử tưởng tượng chúng ta có một thuộc tính computed **A** có nhiều thao tác tính toán trên một mảng dữ liệu lớn. Chúng ta lại có nhiều thuộc tính computed phụ thuộc vào **A**. Nếu không cache lại, chúng ta phải thực thi hàm getter của **A** nhiều lần! Trong trường hợp không cần sử dụng cache, hãy sử dụng phương thức thay cho computed.

### Thuộc tính computed và watched

Vue cung cấp một cách khái quát hơn để quan sát và phản hồi những thay đổi trên dữ liệu đó là: **thuộc tính watch**. Khi bạn cần có những dữ liệu được thay đổi dựa trên sự thay đổi của những dữ liệu khác, trong trường hợp đó có lẻ bạn đang lạm dụng `watch` - đặc biệt hơn nếu bạn đã từng sử dụng AngularJS. Tuy nhiên, những trường hợp đó tốt hơn là nên dùng computed thay cho `watch`. Hãy xem xét ví dụ sau đây:

``` html
<div id="demo">{{ fullName }}</div>
```

``` js
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Phu',
    lastName: 'Ba',
    fullName: 'Phu Ba'
  },
  watch: {
    firstName: function (val) {
      this.fullName = val + ' ' + this.lastName
    },
    lastName: function (val) {
      this.fullName = this.firstName + ' ' + val
    }
  }
})
```

Đoạn code phía trên thì imperative và lập lại. Hãy so sánh với cách sử dụng computed:

``` js
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Phu',
    lastName: 'Ba'
  },
  computed: {
    fullName: function () {
      return this.firstName + ' ' + this.lastName
    }
  }
})
```

Cách này tốt hơn nhiều đúng không?

### Computed Setter

Những thuộc tính computed mặc định chỉ có getter, nhưng bạn cũng có thể cung cấp setter nếu cần thiết:

``` js
// ...
computed: {
  fullName: {
    // getter
    get: function () {
      return this.firstName + ' ' + this.lastName
    },
    // setter
    set: function (newValue) {
      var names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
// ...
```

Bây giờ, khi bạn gán `vm.fullName = 'Phu Ba'`, thì setter sẽ được gọi, lúc này `vm.firstName`, `vm.lastName` sẽ được cập nhật lại dựa vào những thay đổi trong setter.

## Watcher

Thuộc tính computed thích hợp cho hầu hết các trường hợp, nhưng cũng có lúc cần tới những watcher tùy chỉnh. Đó là lí do tại sao Vue cung cấp một cách khai quát hơn để phản ứng (react) lại với việc thay đổi dữ liệu trong qua `watch`. Cách sử dụng này rất hữu ích khi bạn muốn thực hiện những tính toán không đồng bộ và tiếu tốn cho việc thay đổi dữ liệu.

Ví dụ:

``` html
<div id="watch-example">
  <p>
    Hãy hỏi một câu hỏi yes/no:
    <input v-model="question">
  </p>
  <p>{{ answer }}</p>
</div>
```

``` html
<!-- Những thư viện như xử lí ajax, xử lí mảng đã có sẳn, -->
<!-- vì vậy Vue không cố gắng phát minh lại những thứ đó. -->
<!-- Bạn có thể sử dụng thoải mái những thư viện mà bạn thấy quen thuộc -->
<script src="https://cdn.jsdelivr.net/npm/axios@0.12.0/dist/axios.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/lodash@4.13.1/lodash.min.js"></script>
<script>
var watchExampleVM = new Vue({
  el: '#watch-example',
  data: {
    question: '',
    answer: 'Không hỏi thì lấy gì mà trả lời!'
  },
  watch: {
    // bât cứ lúc nào câu hỏi thay đổi, hàm bên dưới sẽ chạy
    question: function (newQuestion) {
      this.answer = 'Đang chờ bạn hỏi xong nè...'
      this.getAnswer()
    }
  },
  methods: {
    // _.debounce là một hàm được cung cấp bởi lodash.
    // Để tìm hiểu rõ hơn cách hoạt động của hàm này,
    // bạn có thể truy cập: https://lodash.com/docs#debounce 
    getAnswer: _.debounce(
      function () {
        if (this.question.indexOf('?') === -1) {
          this.answer = 'Câu hỏi thì phải kết thúc bằng dấu ? nha. ;-)'
          return
        }
        this.answer = 'Đang suy nghĩ...'
        var vm = this
        axios.get('https://yesno.wtf/api')
          .then(function (response) {
            vm.answer = _.capitalize(response.data.answer)
          })
          .catch(function (error) {
            vm.answer = 'Lỗi rồi! Không thể truy cập API. ' + error
          })
      },
      // Sau khi người dùng kết thúc việc gõ câu hỏi,
      // hệ thống sẽ chờ 500 miliseconds rồi mới thực hiện nội dung bên trong hàm
      500
    )
  }
})
</script>
```

Kết quả:

{% raw %}
<div id="watch-example" class="demo">
  <p>
    Hãy hỏi một câu hỏi yes/no:
    <input v-model="question">
  </p>
  <p>{{ answer }}</p>
</div>
<script src="https://cdn.jsdelivr.net/npm/axios@0.12.0/dist/axios.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/lodash@4.13.1/lodash.min.js"></script>
<script>
var watchExampleVM = new Vue({
  el: '#watch-example',
  data: {
    question: '',
    answer: 'Không hỏi thì lấy gì mà trả lời!'
  },
  watch: {
    question: function (newQuestion) {
      this.answer = 'Đang chờ bạn hỏi xong nè...'
      this.getAnswer()
    }
  },
  methods: {
    getAnswer: _.debounce(
      function () {
        var vm = this
        if (this.question.indexOf('?') === -1) {
          vm.answer = 'Câu hỏi thì phải kết thúc bằng dấu ? nha. ;-)'
          return
        }
        vm.answer = 'Đang suy nghĩ...'
        axios.get('https://yesno.wtf/api')
          .then(function (response) {
            vm.answer = _.capitalize(response.data.answer)
          })
          .catch(function (error) {
            vm.answer = 'Lỗi rồi! Không thể truy cập API. ' + error
          })
      },
      500
    )
  }
})
</script>
{% endraw %}

Trong trường hợp này, sử dụng `watch` cho phép chúng ta thực hiện những tính toán không đồng bộ (ví dụ: truy cập tới một API) và giới hạn những tính toán đó, sau đó giá trị sẽ được gán ngay lập tức vào state khi có được đáp án cuối cùng. Những điều trên sẽ không thực hiện được nếu bạn sử dụng thuộc tính computed.

Những thông tin thêm về `watch` có thể được tìm thấy tại [vm.$watch API](../api/#vm-watch).
