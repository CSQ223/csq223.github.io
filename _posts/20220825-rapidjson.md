# 1.初始化一个Document对象
```
Document d;
```
参数：无
# 2.解析一个 JSON 字符串至一个 document (DOM)
```
GenericDocument& Parse(const Ch* str)
//使用 d.Parse(str.c_str());
```
参数：1) str.c_str() :json字符串
# 3.对 DOM 作出修改
```
Value& s1 = d["age"];
s.SetString("json库");
```
参数：1) str.c_str() :json字符串
# 4.修改json文件结构
```
GenericValue& SetInt(int i) { this->~GenericValue(); new (this) GenericValue(i); return *this; }
GenericValue& SetUint(unsigned u) { this->~GenericValue(); new (this) GenericValue(u); return *this; }
GenericValue& SetInt64(int64_t i64) { this->~GenericValue(); new (this) GenericValue(i64); return *this; }
GenericValue& SetUint64(uint64_t u64) { this->~GenericValue(); new (this) GenericValue(u64); return *this; }
GenericValue& SetDouble(double d) { this->~GenericValue(); new (this) GenericValue(d); return *this; }
GenericValue& SetFloat(float f) { this->~GenericValue(); new (this) GenericValue(static_cast<double>(f)); return *this; }
```
# 5.获取json文
```
int64_t GetInt64() const { RAPIDJSON_ASSERT(data_.f.flags & kInt64Flag); return data_.n.i64; }
uint64_t GetUint64() const { RAPIDJSON_ASSERT(data_.f.flags & kUint64Flag); return data_.n.u64; }
unsigned GetUint() const { RAPIDJSON_ASSERT(data_.f.flags & kUintFlag); return data_.n.u.u; }
int GetInt() const { RAPIDJSON_ASSERT(data_.f.flags & kIntFlag); return data_.n.i.i; }
```

# 6.把 DOM 转换（stringify）至 JSON 字符串
```
StringBuffer buffer;
Writer<StringBuffer> writer(buffer);
d.Accept(writer);
```
初始化StringBuffer ，writer类，使用bool Accept(Handler& handler) const方法将DOM转换至json字符串
