### 请求压缩:
Spring cloud Feign 支持请求和响应压缩，减少通信中的性能损耗:
feign.compression.request.enabled=true;
feign.compression.response.enabled=true;
更细致的配置:
feign.compression.request.mime-types = text/xml,application/xml,application/json
feign.compression.request.min-request-size=2048