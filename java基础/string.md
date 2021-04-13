## 主要方法

- substring：截取字符串（左闭右开）
- split：根据给定的正则表达式分割字符，返回string数组
- trim：去左右空格
- replace：新字符替换旧字符。也支持charSequence，也是替换全部，但不支持正则表达式
- replaceAll：用新字符串替代正则表达式匹配的所有字符
- replaceFirst：基于正则表达式替换第一个
- charAt：指定位置的字符
- toCharArray：转为char数组
- length：长度
- indexOf(string s, int index)：获取字符串从index开始的第一个s的下标
- lastIndexOf(string s, int index)：index之前最后一个s的下标
- toUpCase，toLoewrCase：转换大小写6
- equals：判断字符串是否相等，而非引用是否相等
- equalsIgnoreCase：忽略大小写判断是否一样
- contains：判断字符串里是否包含指定内容
- startWith/
- endWith





循环多次操作才用stringBuilder，否则直接拼接效率差不多，多次效率低是因为创建了多个stringBuilder

