## Object.java
```java
private static native void registerNatives();
static {
    registerNatives();
}
```
一开始就有上面的代码，所以现在看来，这个底层的本地方法还是不知道怎么去查看。所以现在就是一直在找方法，有一个就是直接下载opensdk，然后去查看，但是现在还是没有找到。
网上找到了一个方法，下载了openSDK的包，然后终于是在下面的文件中找到了方法
`openjdk\jdk\src\share\native\java\lang`
其中Object相关的本地方法是：
```c
#include <stdio.h>
#include <signal.h>
#include <limits.h>

#include "jni.h"
#include "jni_util.h"
#include "jvm.h"

#include "java_lang_Object.h"

static JNINativeMethod methods[] = {
    {"hashCode",    "()I",                    (void *)&JVM_IHashCode},
    {"wait",        "(J)V",                   (void *)&JVM_MonitorWait},
    {"notify",      "()V",                    (void *)&JVM_MonitorNotify},
    {"notifyAll",   "()V",                    (void *)&JVM_MonitorNotifyAll},
    {"clone",       "()Ljava/lang/Object;",   (void *)&JVM_Clone},
};

JNIEXPORT void JNICALL
Java_java_lang_Object_registerNatives(JNIEnv *env, jclass cls)
{
    (*env)->RegisterNatives(env, cls,
                            methods, sizeof(methods)/sizeof(methods[0]));
}

JNIEXPORT jclass JNICALL
Java_java_lang_Object_getClass(JNIEnv *env, jobject this)
{
    if (this == NULL) {
        JNU_ThrowNullPointerException(env, NULL);
        return 0;
    } else {
        return (*env)->GetObjectClass(env, this);
    }
}
```
从上面C的方法中，我们可以知道本地registerNative方法对应的就是Java_java_lang_Object_registerNatives(JNIEnv *env, jclass cls)方法，针对于其中的方法，我们在进行研究，是调用了JNIEnv *evn的方法，现在 的问题就是我们需要试着去找到对应的JNIEnv的引用是在哪里。
从文件中，我们可以看到有一些头文件，首先，我试着把这些头文件都找出来看下。
其中stdio.h是一个标准的输入输出文档，这边我们就不再进行详细的说明，可以参考[百度百科stdio.h](https://baike.baidu.com/item/stdio.h/9637809?fr=aladdin), 现在我是想知`(*env)->RegisterNatives(env, cls,methods, sizeof(methods)/sizeof(methods[0]));`具体是怎么实现的，怎么做的，注册本地方法是什么意思呢，也就是被本地的整个类给注册进去了吗，也就是每个java对象都会有这样的操作，
