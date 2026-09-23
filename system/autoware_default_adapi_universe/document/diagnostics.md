<a id="diagnostics"></a>

# 诊断

## /adapi/node/localization: state

当定位状态为 INITIALIZED 时，诊断级别为 OK；否则为 ERROR。

## /adapi/node/routing: state

当路线状态为 SET、REROUTING 或 ARRIVED 时，诊断级别为 OK；否则为 ERROR。

## /adapi/node/mrm_request: delegate

未请求 delegate 策略时，诊断级别为 OK；否则为 ERROR。
将 delegate 策略的发送方设置为键。
