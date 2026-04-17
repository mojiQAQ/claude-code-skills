---
name: cursor-billing
description: Cursor 订阅报销自动化处理。自动登录多个 Cursor 账户，下载收据 PDF，OCR 识别交易记录截图，智能匹配收据与交易记录，统一重命名文件，汇总报销金额。
license: MIT
triggers:
  - /cursor-billing
  - /cursor报销
  - /报销处理
---

# Cursor 订阅报销自动化处理

这个 skill 自动完成 Cursor 订阅的月度报销全流程：登录账户 → 下载收据 → OCR识别交易截图 → 智能匹配 → 文件重命名 → 金额汇总。

## 使用方式

用户需提供以下信息：

1. **工作目录路径**：存放交易记录截图的文件夹
2. **处理月份**：如 2026年2月
3. **账号密码列表**，格式：
```
账户名1: email@example.com / 密码
账户名2: email2@example.com / 密码
...
```

## 完整处理流程

### 第一步：逐个账户登录并下载收据

对每个账户依次执行以下操作：

#### 1.1 登录 Cursor
1. 使用 Playwright 导航到 `https://cursor.com/cn/settings`
2. 点击页面上的"登录"按钮
3. 输入邮箱地址，点击继续
4. 输入密码并提交

#### 1.2 处理 Cloudflare 验证
如遇 Cloudflare 人机验证，使用以下代码点击验证框：
```javascript
async (page) => {
  const frames = page.frames();
  for (const frame of frames) {
    if (frame.url().includes('challenges.cloudflare.com')) {
      const box = await frame.locator('body').boundingBox();
      if (box) {
        await page.mouse.click(box.x + 30, box.y + 20);
      }
    }
  }
}
```
注意：Cloudflare 验证可能需要多次点击，或请用户手动完成。

#### 1.3 下载收据
1. 登录成功后导航到 `https://cursor.com/cn/dashboard?tab=billing`
2. 点击月份选择器，选择目标月份
3. 页面会显示该月的所有发票列表
4. 对每张发票，点击 "View" 按钮打开详情页
5. 在详情页点击 "下载收据" 按钮下载 PDF
6. PDF 默认下载到 Playwright 的下载目录（通常是 `.playwright-mcp/`）
7. 将 PDF 移动到用户的工作目录
8. 重命名为：`{账户名}_Receipt-{编号}_{金额}USD_{YYYYMMDD}.pdf`

#### 1.4 登出并切换账户
处理完当前账户后，登出当前账户，然后登录下一个账户重复以上步骤。

---

### 第二步：OCR 识别交易记录截图

用户的工作目录中会有银行/信用卡的交易记录截图（通常是 PNG 格式）。

使用 macOS 原生 Vision 框架进行 OCR 识别：

```bash
/usr/bin/swift -e "
import Vision
import AppKit

let url = URL(fileURLWithPath: \"图片路径\")
guard let image = NSImage(contentsOf: url),
      let cgImage = image.cgImage(forProposedRect: nil, context: nil, hints: nil) else { exit(1) }

let request = VNRecognizeTextRequest { request, error in
    guard let observations = request.results as? [VNRecognizedTextObservation] else { return }
    for observation in observations {
        if let text = observation.topCandidates(1).first?.string {
            print(text)
        }
    }
}
request.recognitionLanguages = [\"zh-Hans\", \"en\"]
request.recognitionLevel = .accurate

let handler = VNImageRequestHandler(cgImage: cgImage, options: [:])
try? handler.perform([request])
"
```

也可以直接使用 Read 工具读取 PNG 图片（Claude 支持多模态识别图片内容）。

从识别结果中提取以下关键信息：
- **原交易金额**（美元，如 $20.00）
- **交易时间**（精确到日）
- **人民币金额**（如 ¥140.04）
- **卡号尾号**（用于区分账户归属，如尾号 6373 或 2204）

---

### 第三步：智能匹配收据与交易记录

根据以下规则将收据 PDF 与交易记录截图配对：

#### 匹配优先级：
1. **唯一金额直接匹配**：非整数或唯一金额（如 $3.71, $8.98, $11.55, $20.77, $40.01, $60.33, $80.11）可以直接通过金额匹配
2. **金额+日期联合匹配**：相同金额（如多笔 $20.00）需要结合交易日期区分
3. **卡号辅助确认**：通过交易记录中的卡号尾号确认账户归属

#### 卡号与账户对应关系（需根据实际情况更新）：
- 卡号尾号 **6373**：fpc_cursor005, fpc_cursor006
- 卡号尾号 **2204**：fpc_cursor002, fpc_cursor003, fpc_cursor004, fpc.cursor001

---

### 第四步：重命名交易记录图片

将交易记录截图统一重命名为：
```
{账户名}-{金额}USD-{YYYYMMDD}-transaction.png
```

示例：
```bash
mv "img_001.png" "fpc_cursor006-20.00USD-20260104-transaction.png"
```

---

### 第五步：汇总报销金额

输出完整的汇总表格：

#### 匹配明细表
| 账户 | 收据文件 | 交易记录图片 | 美元金额 | 人民币金额 | 日期 |
|------|---------|-------------|---------|-----------|------|
| fpc_cursor006 | fpc_cursor006_Receipt-xxx_20.00USD_20260104.pdf | fpc_cursor006-20.00USD-20260104-transaction.png | $20.00 | ¥140.04 | 01-04 |

#### 汇总表
| 项目 | 金额 |
|------|------|
| 美元总计 | $xxx.xx |
| 人民币总计 | ¥x,xxx.xx |
| 收据数量 | xx 张 |
| 账户数量 | x 个 |

---

## 文件命名规范

| 文件类型 | 命名格式 | 示例 |
|---------|---------|------|
| 收据 PDF | `{账户}_Receipt-{编号}_{金额}USD_{YYYYMMDD}.pdf` | `fpc_cursor006_Receipt-8801-7716_20.00USD_20260104.pdf` |
| 交易记录 | `{账户}-{金额}USD-{YYYYMMDD}-transaction.png` | `fpc_cursor006-20.00USD-20260104-transaction.png` |

日期格式统一为 **YYYYMMDD**（如 20260110）。

---

## 注意事项

1. **Cloudflare 验证**：部分登录可能触发 Cloudflare 人机验证，可能需要多次点击或请用户手动完成
2. **文件移动**：Playwright 下载的文件默认在 `.playwright-mcp/` 目录，需移动到工作目录
3. **OCR 环境**：OCR 使用 macOS Vision 框架，仅支持 macOS 系统
4. **截图准备**：交易记录截图需用户提前放入工作目录
5. **金额精度**：注意美元金额的小数点精度，确保匹配准确
6. **多次扣款**：同一账户可能有多笔扣款（如月中升级/降级），需全部处理
