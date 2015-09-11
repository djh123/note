###jni调用第三方静态库

写错了  这样编译是通过了，可是在实际运用中，却无法访问静态库中的函数的

查看JNi的sample中，模仿写法如下：
```
include $(CLEAR_VARS)     
LOCAL_MODULE    := libA    
LOCAL_SRC_FILES := libA.a       
include $(PREBUILT_STATIC_LIBRARY)  
  
include $(CLEAR_VARS)     
LOCAL_MODULE    := libB       
LOCAL_SRC_FILES := libB.a     
include $(PREBUILT_STATIC_LIBRARY)  
  
include $(CLEAR_VARS)     
LOCAL_MODULE    := libC  
LOCAL_SRC_FILES := libC.a     
include $(PREBUILT_STATIC_LIBRARY)  
  
include $(CLEAR_VARS)             
LOCAL_MODULE    := Test  
LOCAL_SRC_FILES := Test.c        
LOCAL_STATIC_LIBRARIES := libA libB libC     
LOCAL_LDLIBS := -llog  
include $(BUILD_SHARED_LIBRARY)  
```

动态库 没有验证好像是差不多的

```
LOCAL_PATH    := $(call my-dir)  
#  
include $(CLEAR_VARS)     
LOCAL_MODULE    := libmmm    
LOCAL_SRC_FILES := libaaa.so     
include $(PREBUILT_SHARED_LIBRARY)    
  
include $(CLEAR_VARS)  
LOCAL_MODULE    := libtest  
LOCAL_SRC_FILES := com_lp_jni_JMedia.c  
  
LOCAL_LDLIBS    += -L$(SYSROOT)/usr/lib -llog  
  
LOCAL_SHARED_LIBRARIES := libmmm  
  
  
include $(BUILD_SHARED_LIBRARY)  
```