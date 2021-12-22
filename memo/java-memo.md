# javaメモ書き

## Java Access Bridge, JAB, Java Accessibility API

### jabswitchソースコード

* jdk6, jdk7  
jabswitch.cpp無し

* jdk8  
https://github.com/openjdk/jdk8u/blob/master/jdk/src/windows/native/sun/bridge/jabswitch.cpp

* jdk15  
https://github.com/openjdk/jdk15u/blob/master/src/jdk.accessibility/windows/native/jabswitch/jabswitch.cpp

* jdk18  
https://github.com/openjdk/jdk18u/blob/master/src/jdk.accessibility/windows/native/jabswitch/jabswitch.cpp

JDK8～JDK18間で特に差異はない模様。途中でメモリ開放の修正が入ってるくらい。
（コンパイルオプションは未確認）


#### 処理内容メモ書き

`jabswitch.exe -enable`は以下二点の更新を行う。

* ファイル
    * %USERPROFILE%\\.accessibility.properties
* レジストリ
    * HKEY_CURRENT_USER\Software\Microsoft\Windows NT\CurrentVersion\Accessibility

##### ファイル

ファイルの更新処理は、エラーチェック＆エラー通知をしっかり行っており、以下文字列の先頭 "#" を外す動き。  
ソースコード内のコメントでも記載あるけど。

* assistive_technologies=com.sun.java.accessibility.AccessBridge
* screen_magnifier_present=true

```cpp
FILE* origFile;
FILE* tempFile;

void enableJAB() {
    char jabLine[] = "assistive_technologies=com.sun.java.accessibility.AccessBridge\n";
    // 中略
    while (!feof(origFile)) {
        if (fgets(line, 512, origFile) != NULL) {
            if (_stricmp(line, "#assistive_technologies=com.sun.java.accessibility.AccessBridge\n") == 0) {
                fputs(jabLine, tempFile);
            // 以下略
```

##### レジストリ

1. レジストリキー `HKCU\Software\Microsoft\Windows NT\CurrentVersion\Accessibility` の存在有無を確認
    1. 存在しなければそのまま終了（特にエラー通知はしない）
        ```cpp
        // エントリポイント
        // regEnable()の戻り値チェックを行っていない
        void main(int argc, char* argv[]) {
            // 中略
            if( !isXP() )
                regEnable();
            // 中略
        }

        // jabswitch -enable時に呼ばれるレジストリへの登録用関数
        int regEnable() {
            // 中略
            LSTATUS err;
            err = RegOpenKeyEx(HKEY_CURRENT_USER, ACCESSIBILITY_USER_KEY, NULL, KEY_READ|KEY_WRITE, &hKey);
            if (err == ERROR_SUCCESS) {
                // 中略
            }
            return err;
        }
        ```
1. 値の名前: `Configuration` の存在有無を確認
    1. 存在しなければそのまま終了（特にエラー通知はしない）
        ```cpp
        // コードは割愛
        ```
1. 値のデータに元の文字列 + ",oracle_javaaccessbridge"を付与
    ```cpp
    static LPCTSTR STR_ACCESSBRIDGE =
    _T("oracle_javaaccessbridge");
    // 中略
    wsprintf(newStr, L"%s,%s", dataBuffer, STR_ACCESSBRIDGE);
    RegSetValueEx(hKey, ACCESSIBILITY_CONFIG, 0, REG_SZ, (BYTE *)newStr, dataLength);
    ```

### accessibility.properties参照箇所

#### JDK6

jabswitchで参照していた `%USERPROFILE%\\.accessibility.properties` については、JDK6時点でもファイルパス変更はされていないようだ。  
そして %JAVA_HOME%\\lib\\accessibility.properties でもかまわないようだ。

jdk\\src\\share\\classes\\java\\awt\\Toolkit.java
```java
private static void initAssistiveTechnologies() {
    // 中略
    final String sep = File.separator;
    // 中略

    // Try loading the per-user accessibility properties file.
    try {
        File propsFile = new File(
            System.getProperty("user.home") +
            sep + ".accessibility.properties");
        FileInputStream in =
            new FileInputStream(propsFile);

        // Inputstream has been buffered in Properties class
        properties.load(in);
        in.close();
    } catch (Exception e) {
        // Per-user accessibility properties file does not exist
    }

    // Try loading the system-wide accessibility properties
    // file only if a per-user accessibility properties
    // file does not exist or is empty.
    if (properties.size() == 0) {
        try {
            File propsFile = new File(
                System.getProperty("java.home") + sep + "lib" +
                sep + "accessibility.properties");
    // 以下略
```
