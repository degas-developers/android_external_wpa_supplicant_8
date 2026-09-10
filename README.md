# wpa_supplicant_8 for MediaTek gen4m

Fork of [LineageOS/android_external_wpa_supplicant_8](https://github.com/LineageOS/android_external_wpa_supplicant_8),
branch `lineage-23.2`, with one commit on top. Used by the Xiaomi 14T
(`degas`, MT6897) tree, but the fix applies to anything running MediaTek's
`wlan_drv_gen4m`.

## What's changed

`nl80211: do not advertise WPA_VERSION_3 for SAE on MTK gen4m` — five lines
across `driver_nl80211.c` and `Android.bp`.

gen4m doesn't parse `NL80211_WPA_VERSION_3` in a connect request. The AKM
suite comes out empty (`u4AkmSuite=0x0`), `rsnIsSuitableBSS` drops the SAE
candidate, and the join fails with status 16 — so no WPA3 or WPA3-transition
AP will connect.

The commit puts the `WPA_VERSION_3` advertisement behind
`CONFIG_DRIVER_NL80211_MTK`, the same way the tree already exempts BRCM and
SYNA, and adds that define to `wpa_supplicant_driver_cflags_default` so it
actually reaches the vendor supplicant binary. With `WPA_VERSION_2` the driver
reads the AKM correctly and SAE completes over external-auth.

On a 14T without the define: `WPA Versions 0x4`, association rejected with
status 16. With it: `WPA Versions 0x2`, link up as WPA3-SAE.

`wpa_supplicant/nl80211_driver` looks like the soong variable to use for this,
but it only feeds `wpa_supplicant_cflags_defaults`, which no module
references — the guard would never get compiled in.

## Enabling it

In the device's `BoardConfig.mk`:

```make
$(call soong_config_set_bool,wpa_supplicant_8,board_wlan_mtk_gen4m,true)
```

Without that flag the build is identical to upstream.

## Upstream

Submitted to LineageOS Gerrit as change 494457 and abandoned by its author, so
it never landed. That's why this repo exists.

## Rebasing

The branch is rebased onto upstream, not merged, so it stays upstream plus our
one commit. Current base: `bad166e4`, `0dbf8c54` (`lineage-23.2`). Worth
keeping it that way if you fork it further.

## License

BSD, unchanged from upstream — see `README` (`LICENSE` is a symlink to it),
`COPYING`, `NOTICE` and `MODULE_LICENSE_BSD_LIKE`. None of those files were
touched here.
