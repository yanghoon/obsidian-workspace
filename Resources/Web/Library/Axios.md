## on Vue3

```javascript
import { createApp } from 'vue'
import App from './App.vue'
import axios from 'axios'

const app = createApp(App)
app.config.globalProperties.$axios = axios
```

# References

## Vue

* https://velog.io/@kungsboy/Vue3-axios-%EC%84%A4%EC%B9%98-%EB%B0%8F-%EC%82%AC%EC%9A%A9