---
tags: [tool, observability, gpu]
aliases: ["NVIDIA DCGM Exporter"]
---

# DCGM Exporter

NVIDIA DCGM(Data Center GPU Manager)이 수집하는 GPU 하드웨어 지표를 [[Prometheus]]가 긁어갈 수 있는 `/metrics` 포맷으로 노출해주는 익스포터입니다. [[vLLM]] 자체가 노출하는 `vllm:gpu_cache_usage_perc`·`time_to_first_token_seconds` 같은 지표는 어디까지나 **서빙 소프트웨어 관점**의 신호라, GPU 온도·전력·ECC 에러·XID 에러 코드 같은 **하드웨어 관점**의 신호는 별도로 봐야 합니다.

이 구분이 중요한 이유는 "서빙이 느려졌다"는 증상이 소프트웨어 병목(배치가 꽉 참, KV 캐시 부족)과 하드웨어 병목(열 스로틀링으로 GPU 클럭이 강제로 낮아짐)에서 똑같이 나타날 수 있기 때문입니다. vLLM 메트릭만 보면 둘을 구분할 수 없고, DCGM Exporter가 노출하는 GPU 온도·전력 소비 지표를 같이 봐야 "지금 배치가 밀린 건지, GPU가 뜨거워서 스스로 느려진 건지"를 가릴 수 있습니다.

## 카오스 엔지니어링과의 접점

[[카오스 엔지니어링]]에서 다룬 것처럼, DCGM에는 오류 신호를 인위적으로 주입하는 기능이 있어 "실제 하드웨어 장애가 아니라 모니터링·알람 파이프라인이 오류 신호에 제대로 반응하는지"를 검증하는 용도로 쓸 수 있습니다. 반대로 실제 하드웨어 장애(GPU가 버스에서 이탈하는 등)는 커널이 `dmesg`에 남기는 XID 에러 코드로 사후 진단하는 경우가 많은데, 이 XID 코드도 DCGM Exporter를 통해 Prometheus 메트릭으로 노출시켜두면 Grafana 대시보드에서 "GPU 하드웨어 이상"을 소프트웨어 메트릭과 같은 화면에서 추적할 수 있습니다.

## 관련
- [[Prometheus]] — DCGM Exporter가 실제로 데이터를 넘기는 대상
- [[vLLM]] — 소프트웨어 관점 메트릭을 이미 노출하고 있는, 상호 보완 관계의 대상
- [[카오스 엔지니어링]] — DCGM의 오류 주입 기능과 XID 에러 사후 진단이 맞닿는 지점
- [[GPU 선택]] — 어떤 하드웨어를 쓰느냐에 따라 봐야 할 DCGM 지표 항목이 달라짐
