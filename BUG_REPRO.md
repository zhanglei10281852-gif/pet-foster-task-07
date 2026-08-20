# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

客服拿到一条创建宠物失败的响应，响应头里有 `X-Request-ID`，但 JSON 错误体的 `requestId` 是空值，前端错误弹窗和后端日志无法用同一个编号关联。这次仅定位问题而不改代码，生产代码、测试和配置都保持原样；请沿请求上下文、响应封装与错误映射查清编号丢失的位置，并给出能对应这次响应的运行证据。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/pet-foster-task-07
- 仓库地址：https://github.com/zhanglei10281852-gif/pet-foster-task-07.git
- parent SHA：5ccf0c0916147b3d8a146be9e7b4c1ca197dc3ee

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/pet-foster-task-07.git bug-repro
cd bug-repro
git checkout --detach 5ccf0c0916147b3d8a146be9e7b4c1ca197dc3ee
go test ./internal/pet -run ^TestAnnotationErrorEnvelopeKeepsRequestID$ -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/pet -run ^TestAnnotationErrorEnvelopeKeepsRequestID$ -count=1
2026/08/20 15:38:32 INFO pet http request request_id=annotation-request-7 method=POST path=/api/user/login status=400 duration_ms=0
--- FAIL: TestAnnotationErrorEnvelopeKeepsRequestID (0.28s)
    annotation_pet_behavior_test.go:163: status=400 body={"code":400,"message":"pet request validation failed: username and password are required","data":null}
FAIL
FAIL	github.com/zhanglei10281852-gif/pet-foster-go/internal/pet	0.298s
FAIL

```

stderr：

```text
(empty)
```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/pet -run ^TestAnnotationErrorEnvelopeKeepsRequestID$ -count=1
2026/08/20 15:52:46 INFO pet http request request_id=annotation-request-7 method=POST path=/api/user/login status=400 duration_ms=15
--- FAIL: TestAnnotationErrorEnvelopeKeepsRequestID (0.53s)
    annotation_pet_behavior_test.go:163: status=400 body={"code":400,"message":"pet request validation failed: username and password are required","data":null}
FAIL
FAIL	github.com/zhanglei10281852-gif/pet-foster-go/internal/pet	0.759s
FAIL

```

stderr：

```text
(empty)
```

## 通过条件

排查结果须指出 internal/pet/http.go 的 writeAPIError，并结合 internal/pet/domain.go 的 APIErrorEnvelope 缺少 RequestID 字段，说明上下文编号只写入响应头而从未进入 JSON 错误体的机制；运行证据应展示同一失败响应中 X-Request-ID 非空而 requestId 为空。生产实现、测试和配置在诊断前后必须完全一致。
