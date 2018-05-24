## Broadcomm WIFI 芯片 NVRAM
>WIFI NVRAM 存储 Country Code、功率、天线配置等参数  
一般 系统源码目录为：`device/samsung/hmd8895/firmware`  
设备中存储目录为：`system/etc/wifi`  
BCM4356 NVRAM 文件为 bcmdhd.cal_4356

### Country Code
>采用 ISO 3166-1 标准，NVRAM 中字段  `ccode=0x5855`  
源码中 `kernel/drivers/net/wireless/brcm80211/brcmsmac/channel.c`  
>```
>/* Indicates whether the country provided is valid to pass to cfg80211 or not.
> returns true if valid; false if not.
> */
>static bool brcms_c_country_valid(const char *ccode)
>{
>	/* only allow ascii alpha uppercase for the first 2 chars. */
>	if (!((0x80 & ccode[0]) == 0 && ccode[0] >= 0x41 && ccode[0] <= 0x5A &&
>	      (0x80 & ccode[1]) == 0 && ccode[1] >= 0x41 && ccode[1] <= 0x5A))
>		return false;
>
>	/* do not match ISO 3166-1 user assigned country codes that may be in the driver table */
>	if (!strcmp("AA", ccode) ||        /* AA */
>	    !strcmp("ZZ", ccode) ||        /* ZZ */
>	    ccode[0] == 'X' ||             /* XA - XZ */
>	    (ccode[0] == 'Q' &&            /* QM - QZ */
>	     (ccode[1] >= 'M' && ccode[1] <= 'Z')))
>		return false;
>
>	if (!strcmp("NA", ccode))
>		return false;
>
>	return true;
>}
>```  
>以上判断得知，0x5855 为无效的 Country Code，则 China (CN) 应为 0x434E  
### 天线控制
>仅支持 MIMO 的芯片才会用到天线控制，在 NVRAM 中字段  
>```
>rxchain=3
>txchain=3
>```
>"3" 表示双天线工作，"1" 和 "2" 分别表示天线1 和 天线2  
源码中 `kernel/drivers/net/wireless/bcmdhd/dhd_custom_sec.c`  
>```
>#ifdef MIMO_ANT_SETTING
>int dhd_sel_ant_from_file(dhd_pub_t *dhd)
>{
>....
>	/* Select Antenna */
>	bcm_mkiovar("txchain", (char *)&ant_val, 4, iovbuf, sizeof(iovbuf));
>	ret = dhd_wl_ioctl_cmd(dhd, WLC_SET_VAR, iovbuf, sizeof(iovbuf), TRUE, 0);
>	if (ret) {
>		DHD_ERROR(("[WIFI_SEC] %s: Fail to execute dhd_wl_ioctl_cmd(): txchain, ret=%d\n",
>			__FUNCTION__, ret));
>		return ret;
>	}
>
>	bcm_mkiovar("rxchain", (char *)&ant_val, 4, iovbuf, sizeof(iovbuf));
>	ret = dhd_wl_ioctl_cmd(dhd, WLC_SET_VAR, iovbuf, sizeof(iovbuf), TRUE, 0);
>	if (ret) {
>		DHD_ERROR(("[WIFI_SEC] %s: Fail to execute dhd_wl_ioctl_cmd(): rxchain, ret=%d\n",
>			__FUNCTION__, ret));
>		return ret;
>	}
>}
