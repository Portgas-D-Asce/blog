# 获取参数

```ty
<script setup lang="ts">
import { useRoute } from "vue-router"
const route = useRoute();
// 获取 get 参数
let id = route.query.id;
</script>
```

