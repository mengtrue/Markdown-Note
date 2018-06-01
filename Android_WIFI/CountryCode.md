## WIFI Country Code

WIFI 使用的是 2.4G 、5G，以及 802.11 ad 使用的 60GHz，这些频段未避免干扰及安全性，WIFI 联盟规定了不同国家地区民用可用的频段，因此根据国家地区的不同，限制 WIFI 可用的频段，因此 WIFI Country Code 应运而生

当前，Android 源码中

- Android N

  WifiManager.setCountryCode(String countryCode, boolean persist)

  persist 若为 TRUE，则会保存在 Settings.Global.WIFI_COUNTRY_CODE，并传递到 WifiNative->Driver->FW

  通过 WifiCountryCode.setReadyForChange() 生效，需要在 wpa_supplicant started state 作用

  另外，开机后优先级为 此API > ro.boot.wificountrycode > WIFI NV

- Android O

  虽保留了 persist，但不再保存在 Settings.Global.WIFI_COUNTRY_CODE