# 模拟 json 数据

```typescript
import axios from 'axios';

axios.get("./data/categories.json").then((res) => {
    article.value = res.data;
});
```