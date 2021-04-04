---
title: Lambda
date: Lambda
tags: Lambda
---

### 集合

#### 复制到另外一个map

```java
            TreeMap param = new TreeMap();
            bodyMap.forEach((k,v)-> param.put(k.toLowerCase(),v));
            param.keySet().removeIf(key->key.equals("auth_signature"));
```

### 拼接到一个字符串

```java
            StringBuffer buffer = new StringBuffer();
            param.forEach((k,v)-> buffer.append(k).append("=").append(v).append("&"));
            String substring = buffer.substring(0, buffer.length() - 1);
```

