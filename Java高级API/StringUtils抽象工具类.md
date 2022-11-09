# StringUtils抽象工具类

> ### 这是String的一个工具类,里面有很多处理字符串的方法

- #### boolean isEmpty(Object str)

  该方法用来判断传入的字符串是否为null和空串

  - ```java
    public static boolean isEmpty(@Nullable Object str) {
        return str == null || "".equals(str);
    }
    ```

- #### boolean hasLength(CharSequence str)

  该方法处理字符串在不为空的前提下判断长度是否大于0

  - ```java
    public static boolean hasLength(@Nullable CharSequence str) {
        return str != null && str.length() > 0;
    }
    public static boolean hasLength(@Nullable String str) {// 不为空不为空串时返回true
        return str != null && !str.isEmpty();
    }
    ```

- #### boolean hasText(CharSequence str)

  该方法用来判断在不为null,长度大于0,且包含传入的文本内容

  - ```java
    public static boolean hasText(@Nullable CharSequence str) {
        return str != null && str.length() > 0 && containsText(str);
    }
    public static boolean hasText(@Nullable String str) {
        return str != null && !str.isEmpty() && containsText(str);
    }
    ```

### ......