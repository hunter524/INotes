# Gson 源码解析

## 使用层面的 API

- GsonBuilder
- TypeToken
- TypeAdapter/InstanceCreator/JsonDeserializer/JsonSerializer
- TypeAdapterFactory

## 解析层面的源码

### JsonElement(基础解析元素抽象)

- JsonArray
- JsonNull
- JsonObject
- JsonPrimitive

### Excluder(解析过滤规则抽象)
