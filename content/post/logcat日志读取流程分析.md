---
title: "logcat日志读取流程分析"
date: 2021-07-06T22:30:30+08:00
draft: false
categories: [android]
---

logcat 通过读 **/dev/socket/logd** 套接字节点，以从 **logd** 进程中获取信息。

在 `/logcat/logcat.cpp` 的 `__logcat` 方法中调用 `android_logger_list_read`

```java
static int __logcat(android_logcat_context_internal* context) {
......
while (!context->stop &&
           (!context->maxCount || (context->printCount < context->maxCount))) {
        struct log_msg log_msg;
        int ret = android_logger_list_read(logger_list, &log_msg);
        if (!ret) {
            logcat_panic(context, HELP_FALSE, "read: unexpected EOF!\n");
            break;
        }
......
```

`android_logger_list_read`是 liblog/logger_read.c 中的接口

```java
LIBLOG_ABI_PUBLIC int android_logger_list_read(struct logger_list* logger_list,
                                               struct log_msg* log_msg) {
  struct android_log_transport_context* transp;
  struct android_log_logger_list* logger_list_internal =
      (struct android_log_logger_list*)logger_list;

  int ret = init_transport_context(logger_list_internal);
  if (ret < 0) {
    return ret;
  }
......
}
```

其中调用了`init_transport_context`，该接口同样位于 liblog/logger_read.c ，主要实现如下

```java
static int init_transport_context(struct android_log_logger_list* logger_list) {
......
__android_log_lock();
  /* mini __write_to_log_initialize() to populate transports */
  if (list_empty(&__android_log_transport_read) &&
      list_empty(&__android_log_persist_read)) {
    __android_log_config_read();
  }
  __android_log_unlock();
......
}
```

其中调用了`__android_log_config_read`，位于 /liblog/config_read.c，主要实现如下

```java
LIBLOG_HIDDEN void __android_log_config_read() {
......
#if (FAKE_LOG_DEVICE == 0)
  if ((__android_log_transport == LOGGER_DEFAULT) ||
      (__android_log_transport & LOGGER_LOGD)) {
    extern struct android_log_transport_read logdLoggerRead;
    extern struct android_log_transport_read pmsgLoggerRead;

    __android_log_add_transport(&__android_log_transport_read, &logdLoggerRead);
    __android_log_add_transport(&__android_log_persist_read, &pmsgLoggerRead);
  }
#endif
}
```

其中调用了`logdLoggerRead`，位于 /liblog/logd_reader.c，主要实现如下

```java
LIBLOG_HIDDEN struct android_log_transport_read logdLoggerRead = {
  .node = { &logdLoggerRead.node, &logdLoggerRead.node },
  .name = "logd",
  .available = logdAvailable,
  .version = logdVersion,
  .read = logdRead,
  .poll = logdPoll,
  .close = logdClose,
  .clear = logdClear,
  .getSize = logdGetSize,
  .setSize = logdSetSize,
  .getReadableSize = logdGetReadableSize,
  .getPrune = logdGetPrune,
  .setPrune = logdSetPrune,
  .getStats = logdGetStats,
};
```

其中通过`logdRead`函数指针为`.read`字段 进行赋值，`logdRead`的主要实现如下

```java
static int logdRead(struct android_log_logger_list* logger_list,
                    struct android_log_transport_context* transp,
                    struct log_msg* log_msg) {
......
  ret = logdOpen(logger_list, transp);
  if (ret < 0) {
    return ret;
  }
......
   ret = recv(ret, log_msg, LOGGER_ENTRY_MAX_LEN, 0);
......
}
```

里面调用了`logdOpen`来打开套接字`logdr`，而后返回`logdRead`进行`recv`。

```java
static int logdOpen(struct android_log_logger_list* logger_list,
                    struct android_log_transport_context* transp) {
......
sock = socket_local_client("logdr", ANDROID_SOCKET_NAMESPACE_RESERVED,
                             SOCK_SEQPACKET);
......
	return sock;
}
```

