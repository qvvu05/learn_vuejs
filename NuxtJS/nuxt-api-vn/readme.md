# asyncData()

asyncdata được gọi mỗi khi load component. Nó có thể được gọi từ server-side hoặc trước khi chuyển đến route tương ứng. Phương thức này nhận `context` làm tham số thứ nhất, bạn có thể sử dụng đó để fetch dữ liệu và trả về component data.

**Lưu ý**: Không có quyền truy cập thuộc tính của component bằng `this` bên trong asyncData bởi vì nó được gọi trước khi khởi tạo component

# fetch
* phương thức fetch thường được dùng để fill the store trước kh render trang. Nó như asyncData ngoại trừ nó không đặt dữ liệu của component*

Phương thức `fetch` nếu được cài đặt sẽ được gọi trước khi mỗi lần load component (**chỉ page component**).
Nó có thể gọi từ server-side howcj trước khi điều hướng đến route tương ứng.

Nếu muốn gọi store action, sử dụng store.dispatch trong fetch, đảm bảo rằng đợi đến khi kết thúc action bằng cách sử dụng async/await:
(dispatch gọi từ component, commit gọi từ store)
```
<script>
export default {
  async fetch ({ store, params }) {
    await store.dispatch('GET_STARS');
  }
}
</script>
```
bên trong store/index.js:
```
// ...
export const actions = {
  async GET_STARS ({ commit }) {
    const { data } = await axios.get('http://my-api/stars')
    commit('SET_STARS', data)
  }
}
```

# middleware
*middleware cho phép bạn định nghĩa một hàm có thể chạy trước khi render page hoặc 1 nhóm các page*

mọi `middleware` cần được đặt trong thư mục `middleware`. Tên của file sẽ là tên của `middleware`

`middleware` nhận context là tham số đầu:
```
export default function (context){
    context.userAgent=context.isServer ? context.req.header['user-agent']: navigator.userAgent
}
```

`middleware` sẽ được thực thi theo thứ tự sau:
* `nuxt.config.js`
* Matched layouts
* Matched pages

Middleware có thể là bất đồng bộ. Để làm được điều đó, chỉ cần trả về hàm Promise hoặc sử dụng hàm `callback`
`middleware/stats.js:`
```
import axios from 'axios'

export default function ({ route }) {
  return axios.post('http://my-stats-api.com', {
    url: route.fullPath
  })
}
```
Trong file `nuxt.config.js:`
```
module.exports = {
  router: {
    middleware: 'stats'
  }
}
```
`middleware stats` sẽ được gọi mỗi khi thay đổi route
*(dùng trong việc xác thực user, bên trong middleware file, sẽ có hàm kiểm tra xem user còn đăng nhập không, nếu không sẽ chuyển sang trang login)*