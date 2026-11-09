# 蓋斯克網路測速 · 專業診斷

一個以 Cloudflare 全球邊緣網路為測量引擎、Firebase 托管的商業級網路測速與診斷工具。除了測出下載、上傳、延遲、抖動與連線穩定度，還會由智慧診斷引擎**指出網路瓶頸在哪裡**、對照台灣與全球平均值，並給出可執行的改善建議。

前端為單一自包含檔案 `public/index.html`，直接部署即可，無需建置流程。

## 功能亮點

- **即時圓形儀表** — 測試過程中速度數字與弧形進度即時跳動
- **六大核心指標** — 下載、上傳、延遲 Ping、抖動 Jitter、連線穩定度（TCP 重傳）、負載延遲，各自標示良好／普通／偏弱
- **依序測量＋平滑進度條** — 延遲 → 下載 → 上傳依序進行（官方預設是下載/上傳交錯，畫面會來回跳），進度條只前進不後退，並顯示預估剩餘時間，可隨時停止
- **Bufferbloat（緩衝膨脹）分析** — 比較「閒置延遲」與「下載／上傳負載時延遲」的增量，並給 A/B/C/D 分級。這是「有人在用網路時全家變慢」的關鍵成因，多數測速工具不會測
- **智慧診斷引擎**（規則式判讀，非 AI 模型）— 自動找出封包重傳、Bufferbloat、高延遲、高抖動、頻寬不足等問題，依嚴重度排序，每項附上白話說明與具體修正建議（例如啟用 SQM／QoS、改用有線）
- **市場平均比較** — 以長條圖對照「您的網路 / 台灣平均 / 全球平均」（業界典型參考值），一眼看出自己落在哪個水準
- **使用情境評估** — 4K 串流、視訊會議、競技遊戲、雲端遊戲、大檔上傳、多人同時使用，各自標示順暢／勉強可用／不建議
- **一鍵存成報告圖片**（html2canvas）與重新測試
- **無線網路現場分析**（給蓋斯克遠端支援用）— Ping 穩定度（含逐次延遲圖與尖峰標示）、回應速度、無線訊號品質、Airtime 頻道壅塞、附近 WiFi 數量、無線連線等級（幾串流），每項標示「實測／推估／客戶回報」
- **現場資訊點選** — 客戶點選上網方式、申裝速度、附近 WiFi 數量、與分享器距離，判讀即時更新（Android Chrome 會自動偵測上網方式）
- **傳給蓋斯克工程師** — 一鍵複製純文字摘要，貼到 LINE 就能遠端判斷

## 網路健康度評分

總分 0–100，由六項指標加權後計算，並套用「短板封頂」機制——只要有關鍵指標明顯偏弱（如 TCP 重傳 > 3%、Bufferbloat > 150ms、延遲 > 120ms、抖動 > 50ms、下載 < 25Mbps），總分會被封頂，避免高頻寬掩蓋掉糟糕的穩定度。

| 分數 | 等級 | 說明 |
| --- | --- | --- |
| 90–100 | 極佳 | 高畫質串流、視訊會議、線上遊戲可同時順跑 |
| 80–89 | 良好 | 適合絕大多數日常與多裝置使用 |
| 60–79 | 普通 | 基本使用足夠，高互動需求時可能偶有延遲 |
| 0–59 | 偏弱 | 有明顯瓶頸，建議依診斷報告改善 |

### 指標評級門檻

| 指標 | 良好 | 普通 | 偏弱 |
| --- | --- | --- | --- |
| 下載速度 | ≥ 95 Mbps | 25–95 Mbps | < 25 Mbps |
| 上傳速度 | ≥ 42 Mbps | 10–42 Mbps | < 10 Mbps |
| 延遲 Ping | ≤ 30 ms | 30–60 ms | > 60 ms |
| 抖動 Jitter | ≤ 10 ms | 10–30 ms | > 30 ms |
| TCP 重傳率 | ≤ 1% | 1–3% | > 3% |
| Bufferbloat（負載延遲增量） | ≤ 30 ms（A） | 30–150 ms（B/C） | > 150 ms（D） |

> **關於丟包**：Cloudflare 的丟包測試需要 TURN 憑證，只開放給 speed.cloudflare.com 自己使用（其他網域會被拒絕，403），所以本工具改讀 Cloudflare 回應附帶的伺服器端 TCP 統計（`Server-Timing: cfL4` 的 `sent`/`retrans`）來估算重傳率；只有偵測到重傳才會列為問題，量不到的項目不計分。

## 無線網路分析的判讀方式

瀏覽器沒有 API 可以讀取 WiFi 訊號強度（RSSI）、天線數、頻道使用率，也不能掃描附近的 WiFi，所以這部分是從測速數據推估，再搭配客戶點選的現場資訊：

| 項目 | 來源 | 判讀依據 |
| --- | --- | --- |
| Ping 穩定度 | 實測 | 約 35 次閒置 ping（測速前、中、後）：抖動、P95 與最低值差距、尖峰比例（高於一般值 +max(25ms, 80%)） |
| 回應速度 | 實測 | 閒置延遲中位數；最快值很低但中位數被拉高時，提示延遲耗在無線端 |
| 無線訊號品質 | 推估 | TCP 重傳率（高速下 1–3% 視為正常）、滿載速度起伏、最高速度、與分享器距離 |
| Airtime 頻道壅塞 | 推估 | 閒置抖動、延遲尖峰、下載速度起伏大但重傳少；客戶回報的 WiFi 數量只在實測有壅塞跡象時加重判斷 |
| 無線連線等級 | 推估 | 實際 TCP 速度約為無線連線速率 60–70% 反推；跑到申裝速度 80% 以上或不知道申裝速度時顯示「至少」 |
| 附近 WiFi 數量 | 客戶回報 | 請客戶看手機「設定 → Wi‑Fi」清單數量 |

> 「台灣平均／全球平均」為業界典型參考值（概念參照 Ookla Speedtest Global Index 固網中位級距與 Cloudflare AIM 指標），供相對定位使用，非即時官方統計。單次測試會有波動，建議不同時段多測幾次取平均。

## 技術棧

- 純 HTML / CSS / JavaScript（單一檔案，無建置流程）
- 測量引擎：[`@cloudflare/speedtest`](https://speed.cloudflare.com) 1.16.0，壓縮後自架於 `public/vendor/`（不依賴第三方 CDN）
- 報告截圖：html2canvas
- Firebase Hosting

## 本機預覽

```sh
cd public
python3 -m http.server 8000
# 開啟 http://localhost:8000
```
（需連外網以載入 Cloudflare 測量引擎與 CDN 套件。）

## 部署

```sh
firebase deploy
```
專案部署於 Firebase Hosting：https://speedtest-75fb9.web.app

# Cloudflare Speedtest

[![NPM package][npm-img]][npm-url]
[![Build Size][build-size-img]][build-size-url]
[![NPM Downloads][npm-downloads-img]][npm-downloads-url]

`@cloudflare/speedtest` is a JavaScript module to measure the quality of a client's Internet connection. It's the measurement engine that powers the Cloudflare speedtest measurement application available at [https://speed.cloudflare.com](https://speed.cloudflare.com).

The module performs test requests against the [Cloudflare](https://www.cloudflare.com/) edge network and relies on the [PerformanceResourceTiming browser api](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceResourceTiming) to extract timing results.
The network connection is characterized by items such as download/upload bandwidth, latency and packet loss.

Please note that measurement results are collected by Cloudflare on completion for the purpose of calculating aggregated insights regarding Internet connection quality.

## Installation

Add this package to your `package.json` by running this in the root of your project's directory:

```sh
npm install @cloudflare/speedtest
```

## Simple Usage

```js
import SpeedTest from '@cloudflare/speedtest';

new SpeedTest().onFinish = results => console.log(results.getSummary());
```

## API reference

`SpeedTest` is a JavaScript Class and should be instantiated with the `new` keyword. It's not required to pass a config object, as all items have default values.

### Instantiation

```js
new SpeedTest({ configOptions })
```

| Config option | Description | Default |
| --- | --- | :--: |
| **autoStart**: *boolean* | Whether to automatically start the measurements on instantiation. | `true` |
| **downloadApiUrl**: *string* | The URL of the API for performing download GET requests. | `https://speed.cloudflare.com/__down` |
| **uploadApiUrl**: *string* | The URL of the API for performing upload POST requests. | `https://speed.cloudflare.com/__up` |
| **turnServerUri**: *string* | The URI of the TURN server used to measure packet loss. | `turn.speed.cloudflare.com:50000` |
| **turnServerUser**: *string* | The username for the TURN server credentials. | - |
| **turnServerPass**: *string* | The password for the TURN server credentials. | - |
| **measurements**: *array* | The sequence of measurements to perform by the speedtest engine. See [below](#measurement-config) for the specific syntax of this option. ||
| **measureDownloadLoadedLatency**: *boolean* | Whether to perform additional latency measurements simultaneously with download requests, to measure loaded latency (during download). | `true` |
| **measureUploadLoadedLatency**: *boolean* | Whether to perform additional latency measurements simultaneously with upload requests, to measure loaded latency (during upload). | `true` |
| **loadedLatencyThrottle**: *number* | Time interval to wait in between loaded latency requests (in milliseconds). | 400 |
| **bandwidthFinishRequestDuration**: *number* | The minimum duration (in milliseconds) to reach in download/upload measurement sets for halting further measurements with larger file sizes in the same direction. | 1000 |
| **estimatedServerTime**: *number* | If the download/upload APIs do not return a server-timing response header containing the time spent in the server, this fixed value (in milliseconds) will be subtracted from all time-to-first-byte calculations. | 10 |
| **latencyPercentile**: *number* | The percentile (between 0 and 1) used to calculate latency from a set of measurements. | 0.5 |
| **bandwidthPercentile**: *number* | The percentile (between 0 and 1) used to calculate bandwidth from a set of measurements. | 0.9 |
| **bandwidthMinRequestDuration**: *number* | The minimum duration (in milliseconds) of a request to consider a measurement good enough to use in the bandwidth calculation. | 10 |
| **loadedRequestMinDuration**: *number* | The minimum duration (in milliseconds) of a request to consider it to be loading the connection. | 250 |
| **loadedLatencyMaxPoints**: *number* | The maximum number of data points to keep for loaded latency measurements. When more than this amount are available, the latest ones are kept. | 20 |

### Attributes

| Attribute | Description |
| --- | --- |
| **results**: *[Results](#results-object)* | Getter of the current [test results](#results-object) object. May yield incomplete values if the test is still running. |
| **isRunning**: *boolean* | Getter of whether the test engine is currently running. |
| **isFinished**: *boolean* | Getter of whether the test engine has finished all the measurements, and the results are considered final. |

### Methods

| Method | Description |
| --- | --- |
| **play()** | Starts or resumes the measurements. Does nothing if the engine is already running or is finished. |
| **pause()** | Pauses the measurements. Does nothing if the engine is already paused or is finished. |
| **restart()** | Clears the current results and restarts the measurements from the beginning. |

### Notification Events

| Event Method | Arguments | Description |
| --- | --- | --- |
| **onRunningChange** | running: *boolean* | Invoked whenever the test engine starts or stops. The current state is included as a function argument. |
| **onResultsChange** | { type: *string* } | Invoked whenever any item changes in the results, usually indicating the completion of a measurement. The type of measurement that changed is included as an info attribute in the function argument. |
| **onFinish** | results: *[Results](#results-object)* | Invoked whenever the test engine finishes all the measurements. The final [results object](#results-object) is included as a function argument. |
| **onError** | error: *string* | Invoked whenever an error occurs during one of the measurements. The error details are included as a function argument. |

### Measurement config

The specific measurements to be performed by the test engine (and their sequence) can be customized using the `measurements` config option. This should be an array of objects, each with a `type` field, plus additional fields specific to that measurement type.

The default set of measurements that is performed by the engine is:

```js
[
  { type: 'latency', numPackets: 1 }, // initial latency estimation
  { type: 'download', bytes: 1e5, count: 1, bypassMinDuration: true }, // initial download estimation
  { type: 'latency', numPackets: 20 },
  { type: 'download', bytes: 1e5, count: 9 },
  { type: 'download', bytes: 1e6, count: 8 },
  { type: 'upload', bytes: 1e5, count: 8 },
  { type: 'packetLoss', numPackets: 1e3, responsesWaitTime: 3000 },
  { type: 'upload', bytes: 1e6, count: 6 },
  { type: 'download', bytes: 1e7, count: 6 },
  { type: 'upload', bytes: 1e7, count: 4 },
  { type: 'download', bytes: 2.5e7, count: 4 },
  { type: 'upload', bytes: 2.5e7, count: 4 },
  { type: 'download', bytes: 1e8, count: 3 },
  { type: 'upload', bytes: 5e7, count: 3 },
  { type: 'download', bytes: 2.5e8, count: 2 }
]
```

Here are the fields available per measurement type:

#### latency

| Field | Required | Description | Default |
| --- | :--: | --- | :--: |
| **numPackets**: *number* | yes | The number of latency GET requests to perform. These requests are performed against the download API with `bytes=0`, and then the round-trip time-to-first-byte timing between `requestStart` and `responseStart` is extracted. | - |

#### download / upload

Each of these measurement sets are bound to a specific file size. The engine follows a ramp-up methodology per direction (download or upload). Whenever there are multiple measurement sets (with increasing file sizes) for a direction, the engine will keep on performing them until it reaches the condition specified by `bandwidthMinRequestDuration`, at which point further sets in the same direction are ignored.

| Field | Required | Description | Default |
| --- | :--: | --- | :--: |
| **bytes**: *number* | yes | The file size to request from the download API, or post to the upload API. The bandwidth (calculated as bits per second, or bps) for each request is calculated by dividing the `transferSize` (in bits) by the request duration (excluding the server processing time). | - |
| **count**: *number* | yes | The number of requests to perform for this file size. | - |
| **bypassMinDuration**: *boolean* | no | Whether the `bandwidthMinRequestDuration` check should be ignored, and the engine is instructed to proceed with the measurements of this direction in any case. | `false` |

#### packetLoss

Packet loss is measured by submitting a set of UDP packets to a WebRTC TURN server in a round-trip fashion, and determining how many packets do not arrive. The submission of these packets can be done in a batching method, in which there's a sleep time in between batches.

| Field | Required | Description | Default |
| --- | :--: | --- | :--: |
| **numPackets**: *number* | no | The total number of UDP packets to send. | 100 |
| **responsesWaitTime**: *number* | no | The interval of time (in milliseconds) to wait after the latest packet reception before determining the measurement as complete, and all non-returned packets as lost. | 5000 |
| **batchSize**: *number* | no | The number of packets in a batch. If this value is higher than `numPackets` there will be only one batch. | 10 |
| **batchWaitTime**: *number* | no | How long to wait (in milliseconds) between batches. | 10 |
| **connectionTimeout**: *number* | no | Timeout for the connection to the TURN server. | 5000 |

### Results object

An instance object used to access the results of the speedtest measurements. The following methods are available on this object:

| Method | Description |
| --- | --- |
| **getSummary()** | Returns a high-level summary object with the computed results from the performed measurements. |
| **getUnloadedLatency()** | Returns the reduced value of the connection latency while at idle. Requires at least one `latency` measurement. |
| **getUnloadedJitter()** | Returns the connection jitter while at idle. Jitter is calculated as the average distance between consecutive latency measurements. Requires at least two `latency` measurements. |
| **getUnloadedLatencyPoints()** | Returns an array with all the latencies measured while at idle. Includes one value per measurement in sequence. |
| **getDownLoadedLatency()** | Returns the reduced value of the connection latency while loaded in the download direction. Requires `measureDownloadLoadedLatency` to be enabled. |
| **getDownLoadedJitter()** | Returns the connection jitter while loaded in the download direction. Requires `measureDownloadLoadedLatency` to be enabled, and at least two loaded latency measurements. |
| **getDownLoadedLatencyPoints()** | Returns an array with all the latencies measured while loaded in the download direction. Includes one value per loaded measurement in sequence. Requires `measureDownloadLoadedLatency` to be enabled. |
| **getUpLoadedLatency()** | Returns the reduced value of the connection latency while loaded in the upload direction. Requires `measureUploadLoadedLatency` to be enabled. |
| **getUpLoadedJitter()** | Returns the connection jitter while loaded in the upload direction. Requires `measureUploadLoadedLatency` to be enabled, and at least two loaded latency measurements. |
| **getUpLoadedLatencyPoints()** | Returns an array with all the latencies measured while loaded in the upload direction. Includes one value per loaded measurement in sequence. Requires `measureUploadLoadedLatency` to be enabled. |
| **getDownloadBandwidth()** | Returns the reduced value of the download bandwidth (in bps). Requires at least one `download` measurement, longer than the `bandwidthMinRequestDuration` threshold. |
| **getDownloadBandwidthPoints()** | Returns an array with all the download measurement results. Each item will include the following fields: `{ bytes, bps, duration, ping, measTime, serverTime, transferSize }`. |
| **getUploadBandwidth()** | Returns the reduced value of the upload bandwidth (in bps). Requires at least one `upload` measurement, longer than the `bandwidthMinRequestDuration` threshold. |
| **getUploadBandwidthPoints()** | Returns an array with all the upload measurement results. Each item will include the following fields: `{ bytes, bps, duration, ping, measTime, serverTime, transferSize }`. |
| **getPacketLoss()** | Returns the reduced value of the measured packet loss ratio (between 0 and 1). Requires a `packetLoss` measurement set. |
| **getPacketLossDetails()** | Returns an object with the details of the packet loss measurement. Includes the following fields: `{ packetLoss, totalMessages, numMessagesSent, lostMessages }`. Requires a `packetLoss` measurement set. |
| **getScores()** | Returns the computed [AIM scores](https://developers.cloudflare.com/fundamentals/speed/aim/) that categorize the quality of the network connection according to use cases such as streaming, gaming or real-time communications. This score is only available after the engine has finished performing all of the measurements. |

[npm-img]: https://img.shields.io/npm/v/@cloudflare/speedtest
[npm-url]: https://npmjs.org/package/@cloudflare/speedtest
[build-size-img]: https://img.shields.io/bundlephobia/minzip/@cloudflare/speedtest
[build-size-url]: https://bundlephobia.com/result?p=@cloudflare/speedtest
[npm-downloads-img]: https://img.shields.io/npm/dt/@cloudflare/speedtest
[npm-downloads-url]: https://www.npmtrends.com/@cloudflare/speedtest
