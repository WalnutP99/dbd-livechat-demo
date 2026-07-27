# DBD Live Chat Demo

หน้าเว็บตัวอย่างสำหรับทดสอบ **Chatwoot Live Chat Widget** บน environment UAT ของ DBD

Flow การทดสอบ:

**ลูกค้า (widget บนหน้าเว็บ) → Chatwoot UAT → next10 Chat Center (`/chat`)**

## สิ่งที่ต้องมี

- เบราว์เซอร์ Chrome / Edge / Safari
- ต่อ VPN หรือเครือข่าย DBD ได้ (เพื่อเข้าถึง `ctcuat.dbd.go.th`)
- Python 3 หรือเครื่องมือ serve static file อื่น ๆ (แนะนำ — ห้ามเปิดไฟล์ด้วย `file://`)

## โครงสร้างโปรเจกต์

| ไฟล์ | คำอธิบาย |
|------|----------|
| `index.html` | หน้า demo ฝัง Chatwoot SDK |
| `README.md` | คู่มือนี้ |

## วิธีรัน

### วิธีที่ 1 — Python (แนะนำ)

```bash
cd dbd-livechat-demo
python3 -m http.server 8090
```

แล้วเปิดเบราว์เซอร์ที่:

```
http://localhost:8090/
```

### วิธีที่ 2 — npx serve

```bash
npx serve . -p 8090
```

### วิธีที่ 3 — VS Code / Cursor Live Server

เปิด `index.html` ด้วย extension Live Server แล้วใช้ URL ที่ extension สร้างให้

> **สำคัญ:** อย่า double-click เปิด `index.html` โดยตรง (`file://`) — browser จะบล็อกการโหลด SDK จาก domain อื่น

## ค่า config ที่ฝังไว้

ค่าเหล่านี้อยู่ใน `index.html`:

| รายการ | ค่า |
|--------|-----|
| Base URL | `https://ctcuat.dbd.go.th/api/live-chat` |
| SDK script | `{BASE_URL}/packs/js/sdk.js` |
| Website token | `qPzJqpHJKE2nLRppSGJxiura` |
| ภาษา | `th` |
| ตำแหน่งปุ่มแชท | มุมขวาล่าง (`position: "right"`) |
| รูปแบบ widget | `standard` |

### ตัวอย่างโค้ดฝัง widget

```html
<script>
  window.chatwootSettings = {
    position: "right",
    type: "standard",
    locale: "th",
  };

  (function (d, t) {
    var BASE_URL = "https://ctcuat.dbd.go.th/api/live-chat";
    var g = d.createElement(t),
      s = d.getElementsByTagName(t)[0];
    g.src = BASE_URL + "/packs/js/sdk.js";
    g.async = true;
    s.parentNode.insertBefore(g, s);
    g.onload = function () {
      window.chatwootSDK.run({
        websiteToken: "qPzJqpHJKE2nLRppSGJxiura",
        baseUrl: BASE_URL,
      });
    };
  })(document, "script");
</script>
```

## ขั้นตอนทดสอบ

1. รัน local HTTP server ตามด้านบน
2. เปิด `http://localhost:8090/` ในเบราว์เซอร์
3. รอให้ปุ่มแชทปรากฏมุมขวาล่าง
4. คลิกปุ่มแชท → **Start Conversation**
5. ส่งข้อความทดสอบ
6. เปิด next10 Chat Center → ตรวจว่าเคสเข้า channel livechat และ agent ตอบกลับได้

## การปรับแต่ง

### เปลี่ยน website token

แก้ค่า `websiteToken` ใน `index.html` ให้ตรงกับ inbox ที่สร้างใน Chatwoot Admin

### เปลี่ยน Base URL

แก้ตัวแปร `BASE_URL` ถ้าต้องการชี้ไป environment อื่น เช่น internal IP ภายใน VPN

### ตัวเลือก `chatwootSettings` ที่ใช้บ่อย

| คีย์ | คำอธิบาย | ตัวอย่าง |
|------|----------|----------|
| `position` | ตำแหน่งปุ่มแชท | `"left"` / `"right"` |
| `type` | รูปแบบ bubble | `"standard"` / `"expanded_bubble"` |
| `locale` | ภาษา widget | `"th"` / `"en"` |
| `launcherTitle` | ข้อความบนปุ่ม | `"แชทกับเรา"` |
| `hideMessageBubble` | ซ่อนปุ่มแชทตอนเริ่มต้น | `true` / `false` |

ดูรายการเต็มได้ที่ [Chatwoot SDK docs](https://www.chatwoot.com/docs/product/channels/live-chat/sdk/setup/)

## แก้ปัญหาเบื้องต้น

| อาการ | แนวทาง |
|-------|--------|
| ไม่มีปุ่มแชท | ใช้ `http://localhost:...` ไม่ใช่ `file://` |
| Console: โหลด `sdk.js` ไม่ได้ | เช็ก VPN / ลอง ping `ctcuat.dbd.go.th` |
| Console: MIME type `text/html` | WAF อาจบล็อก request — ลองใช้ internal gateway แทน public URL |
| `Request Rejected` จาก ctcuat | WAF บล็อกบางไฟล์ static — ติดต่อทีม infra หรือใช้ proxy ภายใน |
| Widget เปิดแต่ไม่มีสไตล์ | hard refresh `Cmd+Shift+R` (Mac) / `Ctrl+Shift+R` (Windows) |
| ส่งข้อความแล้ว next10 ไม่เห็น | แจ้งทีม dev (webhook / channel mapping) |

## หมายเหตุ

- Repo นี้เป็น **static demo** เท่านั้น — ไม่มี proxy server ในตัว
- Website token ใน repo นี้ใช้สำหรับ UAT — อย่านำไปใช้ production โดยไม่ยืนยันกับทีม

## Repository

```
git@github.com:WalnutP99/dbd-livechat-demo.git
```
