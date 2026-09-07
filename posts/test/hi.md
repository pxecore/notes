
---

#### Test 2: Alt Klasörde Dosya (Otomatik Kategori & Harici Görsel Testi)
**Konum:** `posts/networking/xdp-packet-filter.md`  
*(Burada `category:` bilerek yazılmadı; sistemin klasör adından `NETWORKING` kategorisi türetmesini test etmek için).*

```markdown
---
title: "High-Performance Packet Filtering with XDP"
date: "2026-09-06"
tags: [ebpf, xdp, networking, linux]
image: "https://images.unsplash.com/photo-1558494949-ef010cbdcc31?auto=format&fit=crop&w=800&q=80"
---

"Exploring eXpress Data Path (XDP) hooks inside the Linux network driver layer for bare-metal wire-speed packet processing."

# High-Performance Packet Filtering with XDP

XDP allows arbitrary bytecode execution at the lowest possible software layer in the network subsystem, directly inside the device driver before `sk_buff` allocation.

## Architecture & Drop Hook

```c
#include <linux/bpf.h>
#include <bpf/bpf_helpers.h>

SEC("xdp")
int xdp_drop_all(struct xdp_md *ctx) {
    // Drop all incoming traffic at wire speed
    return XDP_DROP;
}

char _license[] SEC("license") = "GPL";
