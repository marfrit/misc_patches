# Volta — Rock 5 ITX+ (RK3588) upstream kernel status & handoff

**Board:** Radxa ROCK 5 ITX+ (RK3588). In the fleet this is **boltzmann** (kernel-dev host, also the
current RK3588 **NPU-driver vendor** host — mind its load before long builds).
**Last upstream sweep:** 2026-07-13 (covered mid-May → 2026-07-13). See also the linux-rockchip ML
cadence note; read the list via the Anubis workaround below.

This doc lets another agent pick up Volta without re-deriving the landscape. TL;DR per topic:

| Topic | Upstream state (mid-2026) | Action for Volta |
|-------|---------------------------|------------------|
| Video output (DP→HDMI "card house") | Landing, actively iterating | **Track + cherry-pick the two series below** |
| Video decode (rkvdec) | H.264/H.265/AV1 merged; VP9 WIP | Present in the marfrit tree already; wire up + test |
| NPU (rocket) | **Mainline since ~6.16 (mid-2025)** | Base is upstream; frontier is Mesa/Teflon + perf |
| RAM / DDR freq scaling | **Nothing upstream** | Vendor-BSP/rkbin-TPL + our own DDR-blob-RE only |

---

## 1. Video output — the Rock 5 ITX HDMI0 "card house" (main finding)

On the Rock 5 ITX, the second HDMI — **HDMI0, next to the headphone jack — is not a native HDMI.** It is
**DP1 → RA620 (a hardware DP-to-HDMI bridge chip) → HDMI0**. The RA620 does the DP2HDMI conversion in
hardware; the kernel just needs the DP controller + PHY + a `simple-bridge` node. Two mainline series
together light this up (neither merged yet as of 2026-07-13):

- **Andy Yan — "Add support for RK3588 DisplayPort Controller"** (v5→v7, 10 patches). The *controller*
  half. Adds: a reusable **DW DPTX support library** (Synopsys DP-TX), **RK3588 DPTX output** in the
  rockchip DRM driver, **`radxa,ra620` simple-bridge** binding + driver, DP0/DP1 DT nodes, and
  **"DP2HDMI enable for ROCK 5 ITX"** (the board patch you want). Scope: single-stream, 1080p + 4K@60
  **YCbCr4:2:0**, no HDCP/audio yet.
- **Sebastian Reichel — "phy: rockchip: usbdp: Clean up the mess"** (VERY active: v9(38)→v12(36) across
  Jul 9–13 2026). The *PHY* half — the USB3/DP combo PHY that physically drives that DP output. Key DP
  bits: **single-lane DP support, DP aux-bridge registration, DP HPD-handling removal.** Andy Yan's
  controller needs this PHY series under it. Rapid revision cadence ⇒ likely merging soon.

**Adjacent, also worth tracking for Volta:**
- **Ciocaltea — "HDMI 2.0 to DW HDMI QP TX"** (v8, 39 patches) — the *native* HDMI (HDMI2 on RK3588):
  scrambling / SCDC / YUV. So both the DP2HDMI path and direct-HDMI are advancing.
- **VOP2 improvements** (~msg 073038+): YUV420/YUV422 color-format support + reset/robustness.

**Handoff action:** once Reichel v12 + Andy Yan v7 land (or to test early), cherry-pick both onto the
Volta kernel branch, enable `ROCKCHIP_DW_DP` + the `radxa,ra620` bridge in the Rock 5 ITX DT, and verify
HDMI0 output. Read the series via `lists.infradead.org/pipermail/linux-rockchip/2026-July/`.

---

## 2. The DP/eDP clock gotcha (learned on the GenBook, applies to Volta)

RK3588 non-HDMI display outputs (eDP via analogix, **and DP via dw_dp**) take their pixel clock from
**`DCLK_VOP2_SRC`** (a divider off the gpll/cpll/v0pll/aupll pool). HDMI reparents its VP dclk to the
dedicated `pll_hdmiphy0/1` (Collabora/Ciocaltea driver work), so HDMI does NOT share that pool.

**Trap:** the local commit **`ce1da8ed5cd1` "clk: rockchip: rk3588: Drop CLK_SET_RATE_PARENT from
DCLK_VOP2_SRC"** (in `marfrit/linux-7.1-rockchip`, authored by mfritsche) prevents the divider from
reprogramming its parent PLL → a single DP/eDP output can't hit an exact pixel clock → `STRM_VALID`
never sets → **black screen** (this is what broke the GenBook eDP; fixed by reverting on branch
`edp-clk-revert-test`). It was written to stop "HDMI disturbing DP pixel timing" in a **dual-output**
case — but HDMI is already isolated onto `pll_hdmiphy`, so on a single-output board the revert is
correct. **If Volta ever runs two non-HDMI outputs at once (e.g. DP + eDP)**, don't re-add the global
flag-drop; instead pin each output's VP to a *dedicated* source PLL via **`assigned-clock-parents`** in
the Rock-5-ITX DT (e.g. one output → v0pll, another → aupll), extending the HDMI dedicated-PLL model.
Full write-up: noether memory `reference_ampere_710_boot.md`.

---

## 3. Video decode (rkvdec)

Mainline `rkvdec` now carries the RK3588 **vdpu381/383** backends (this *is* "rkvdec2"; the old "no
rkvdec2 driver" line is stale). State: **H.264/H.265 merged early 2026, AV1 merged, VP9 WIP**
(Collabora / D.V.A.B. Sarma). The `marfrit/linux-7.1-rockchip` tree already contains all of it **plus**
in-progress VP9-VDPU381 work (the Bin campaign). VA-API-on-top-of-V4L2 works via `libva-v4l2-request`
(mpv/ffmpeg-fourier). Chromium/Brave uses **direct V4L2** (not VA-API) on ARM, but stock Brave still
defaults to the dead-end VA-API path on RK3588 — the working browser is the `chromium-fourier` build.

## 4. NPU (rocket)
Tomeu Vizoso's `accel/rocket` DRM-accel driver is **mainline since ~6.16 (mid-2025)**; userspace in Mesa.
Little linux-rockchip-list traffic (it lives on accel/dri-devel). Frontier = perf + Mesa/Teflon op
coverage — this is the Rosenblatt-campaign territory boltzmann is currently vendoring.

## 5. RAM / DDR
**No upstream RK3588 DDR/dmc/devfreq/DFS driver exists** (patchwork empty for the whole window). DDR
frequency scaling remains vendor-BSP (rkbin TPL / `rk3588-dmc-oc-*` overlays) + the fleet's own
DDR-blob reverse-engineering. Don't expect upstream help here.

---

## How to read the linux-rockchip list (lore is Anubis-gated)
- **`lists.infradead.org/pipermail/linux-rockchip/YYYY-Month/`** — un-gated pipermail mirror (primary).
- **public-inbox git clone:** `git clone --mirror https://lore.kernel.org/linux-rockchip/git/0.git`
  (git endpoint isn't gated; grep offline). **NNTP:** `nntp.lore.kernel.org` (HTTP-only Anubis = bypassed).
- **patchew.org** / **patchwork.kernel.org/project/linux-rockchip** — un-gated patch-series views.
- For arbitrary Anubis sites: a stock headless browser (Playwright) solves the JS PoW automatically — no
  patch needed — or a ~30-line SHA-256 PoW solver + cookie.
