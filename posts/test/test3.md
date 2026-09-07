
---

#### Test 3: Tarih Belirtilmemiş Dosya (Otomatik Commit Tarihi & Uyarı Kutusu Testi)
**Konum:** `posts/cyber/stack-canaries.md`  
*(Burada `date:` bilerek yazılmadı; sistemin Git commit tarihini otomatik çekmesini test etmek için).*

```markdown
---
title: "Bypassing Stack Canaries via Format Strings"
author: "pxecore"
tags: [security, exploit, binary, linux]
---

"A deep dive into stack cookie generation in GCC and techniques for leaking canaries via arbitrary format string read primitives."

# Bypassing Stack Canaries via Format Strings

Stack canaries (SSP) are randomized guard values inserted between local stack variables and the saved frame pointer / return address.

> [!WARNING]
> This analysis is for educational and defensive vulnerability mitigation research only.

### How the Guard Value is Checked

When compiling with `-fstack-protector-all`, GCC inserts a canary check before function exit:

```c
void vulnerable_function(char *input) {
    char buffer[64];
    // If canary != fs:0x28 -> __stack_chk_fail() is invoked
    strcpy(buffer, input);
}
