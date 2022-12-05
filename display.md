# Debugging display not coming on properly

Disable fbdev emulation?


## lsmod display blank

```
[root@ferret ~]# lsmod
Module                  Size  Used by
cbc                    16384  0
des_generic            16384  0
libdes                 24576  1 des_generic
algif_skcipher         16384  0
md5                    16384  0
algif_hash             16384  0
q6asm_dai              28672  1
q6routing             389120  2 q6asm_dai
af_alg                 32768  2 algif_hash,algif_skcipher
q6asm                  28672  1 q6asm_dai
q6adm                  20480  1 q6routing
q6afe_dai              81920  1
snd_q6dsp_common       49152  3 q6asm,q6afe_dai,q6adm
snd_soc_wsa881x        20480  2
regmap_sdw             16384  1 snd_soc_wsa881x
joydev                 24576  0
hid_multitouch         24576  0
gpio_wcd934x           16384  2
panel_edp              36864  0
q6voice_dai            40960  0
snd_soc_wcd934x       274432  1
q6voice                16384  1 q6voice_dai
snd_soc_wcd_mbhc       24576  1 snd_soc_wcd934x
q6afe                  24576  3 q6voice,q6afe_dai,q6adm
soundwire_qcom         24576  1
q6core                 16384  3 q6asm,q6afe,q6adm
q6cvp                  16384  1 q6voice
q6cvs                  16384  0
venus_dec              28672  0
hid_generic            16384  0
q6mvm                  16384  1 q6voice
q6voice_common         16384  4 q6cvp,q6voice,q6mvm,q6cvs
wcd934x                16384  1
venus_enc              32768  0
regmap_slimbus         16384  2 snd_soc_wcd934x,wcd934x
ti_sn65dsi86           24576  1
videobuf2_dma_contig    20480  2 venus_dec,venus_enc
rpmsg_ctrl             16384  0
fastrpc                28672  0
qrtr_smd               16384  0
i2c_hid_of             16384  0
i2c_hid                24576  1 i2c_hid_of
hci_uart               81920  0
btqca                  20480  1 hci_uart
uvcvideo              110592  0
btbcm                  24576  1 hci_uart
videobuf2_vmalloc      16384  1 uvcvideo
venus_core            126976  2 venus_dec,venus_enc
snd_soc_sdm845         16384  0
videobuf2_memops       16384  2 videobuf2_vmalloc,videobuf2_dma_contig
v4l2_mem2mem           36864  3 venus_dec,venus_core,venus_enc
videobuf2_v4l2         32768  4 venus_dec,uvcvideo,venus_enc,v4l2_mem2mem
snd_soc_rt5663         73728  1 snd_soc_sdm845
videobuf2_common       57344  9 videobuf2_vmalloc,videobuf2_dma_contig,venus_dec,videobuf2_v4l2,uvcvideo,venus_core,venus_enc,v4l2_mem2mem,videobuf2_memops
snd_soc_rl6231         16384  1 snd_soc_rt5663
bluetooth             540672  4 btqca,hci_uart,btbcm
videodev              233472  7 venus_dec,videobuf2_v4l2,uvcvideo,videobuf2_common,venus_core,venus_enc,v4l2_mem2mem
snd_soc_qcom_common    16384  1 snd_soc_sdm845
ecdh_generic           16384  1 bluetooth
ecc                    36864  1 ecdh_generic
soundwire_bus          77824  5 regmap_sdw,snd_soc_sdm845,snd_soc_qcom_common,snd_soc_wsa881x,soundwire_qcom
crct10dif_ce           16384  1
mc                     57344  5 videodev,videobuf2_v4l2,uvcvideo,videobuf2_common,v4l2_mem2mem
coresight_stm          20480  0
rtc_pm8xxx             16384  1
qcom_stats             16384  0
ath10k_snoc            53248  0
reset_qcom_pdc         16384  1
stm_core               28672  1 coresight_stm
coresight_funnel       16384  0
coresight_tmc          36864  0
coresight_replicator    16384  0
camcc_sdm845           49152  0
i2c_qcom_geni          20480  0
ath10k_core           491520  1 ath10k_snoc
qcom_rng               16384  0
qcom_q6v5_mss          32768  0
coresight              65536  4 coresight_tmc,coresight_stm,coresight_replicator,coresight_funnel
ipa                   176128  0
ath                    36864  1 ath10k_core
qcom_q6v5_pas          24576  0
mac80211              548864  1 ath10k_core
qrtr                   28672  55 qrtr_smd
qcom_pil_info          16384  2 qcom_q6v5_pas,qcom_q6v5_mss
libarc4                16384  1 mac80211
qcom_q6v5              16384  2 qcom_q6v5_pas,qcom_q6v5_mss
slim_qcom_ngd_ctrl     24576  0
qcom_sysmon            20480  3 qcom_q6v5_pas,qcom_q6v5,qcom_q6v5_mss
icc_osm_l3             16384  0
qcom_wdt               16384  0
qcom_common            16384  5 ipa,ath10k_snoc,qcom_q6v5_pas,slim_qcom_ngd_ctrl,qcom_q6v5_mss
qcom_glink_smem        16384  1 qcom_common
pwm_bl                 16384  0
icc_bwmon              16384  0
cfg80211              380928  3 ath,mac80211,ath10k_core
rfkill                 32768  4 bluetooth,cfg80211
fuse                  135168  1
ip_tables              32768  0
x_tables               40960  1 ip_tables
ipv6                  471040  40
```