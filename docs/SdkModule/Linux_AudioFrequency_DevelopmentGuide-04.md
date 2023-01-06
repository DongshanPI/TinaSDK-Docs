## 4 常用接口说明

这里主要介绍alsa-lib中的常用接口

### 4.1 4.1 control接口

为了方便操作访问，alsa-lib中封装了相关接口,通过control节点(/dev/snd/controlCX)去获取、设置control elements

主要涉及到的接口：

```
snd_ctl_open
snd_ctl_elem_info_get_id
snd_ctl_elem_info_set_id
snd_ctl_elem_info
snd_ctl_ascii_value_parse
snd_ctl_elem_read
snd_ctl_elem_write
snd_ctl_close
```

详细control接口说明请查阅：

https://www.alsa-project.org/alsa-doc/alsa-lib/control.html

https://www.alsa-project.org/alsa-doc/alsa-lib/group___control.html

下面是一个设置音量接口的例子:

```
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <stdint.h>
#include <unistd.h>
#include <string.h>
#include <alsa/asoundlib.h>
#define DEV_NAME "hw:audiocodec"
#define VOLUME_CONTROL "name='LINEOUT volume'"
/* Fuction to convert from percentage to volume. val = volume */
static int convert_volume(int percent, long min, long max)
{
    long range = max - min;
    if (range == 0)
    	return 0;
    return (int)((range * percent / 100) + min);
}

bool controlVolume(int volume_percent)
{
    int err = -1;
    snd_ctl_t *handle = NULL;
    char *card = DEV_NAME;
    char *volume_control = VOLUME_CONTROL;
    char volume_string[4];
    long min, max, raw;
    snd_ctl_elem_info_t *info = NULL;
    snd_ctl_elem_id_t *id = NULL;
    snd_ctl_elem_value_t *control = NULL;
    if (volume_percent > 100 || volume_percent < 0)
    	return false;
    snd_ctl_elem_info_alloca(&info);
    snd_ctl_elem_id_alloca(&id);
    snd_ctl_elem_value_alloca(&control);
    err = snd_ctl_ascii_elem_id_parse(id, volume_control);
    if (err < 0) {
        fprintf(stderr, "Wrong control identifier: %s\n", volume_control);
        goto failed;
    }
    err = snd_ctl_open(&handle, card, 0);
    if (err < 0) {
        fprintf(stderr, "Control device %s open error:%s\n", card, snd_strerror(err));
        goto failed;
    }
    snd_ctl_elem_info_set_id(info, id);
    err = snd_ctl_elem_info(handle, info);
    if (err < 0) {
        fprintf(stderr, "Cannot find the given element from control %s\n", card);
        goto failed;
    }
    snd_ctl_elem_info_get_id(info, id);
    snd_ctl_elem_value_set_id(control, id);
    err = snd_ctl_elem_read(handle, control);
    if (err < 0) {
        fprintf(stderr, "Cannot read the given element from control %s\n", card);
        goto failed;
    }
    min = snd_ctl_elem_info_get_min(info);
    max = snd_ctl_elem_info_get_max(info);
    snprintf(volume_string, sizeof(volume_string), "%d", convert_volume(volume_percent, min
    , max));
    /*printf("set volume %s, [%u%%]\n", volume_string, volume_percent);*/
    err = snd_ctl_ascii_value_parse(handle, control, info, volume_string);
    if (err < 0) {
        fprintf(stderr, "Control %s parse error: %s\n", card, snd_strerror(err));
        goto failed;
    }
    err = snd_ctl_elem_write(handle, control);
    if (err < 0) {
        fprintf(stderr, "Control %s write error: %s\n", card, snd_strerror(err));
        goto failed;
	}
failed:
    if (info)
    	snd_ctl_elem_info_free(info);
    if (id)
    	snd_ctl_elem_id_free(id);
    if (control)
    	snd_ctl_elem_value_free(control);
    if (handle)
    	snd_ctl_close(handle);
    return ((err < 0)? false : true);
}
```

### 4.2 4.2 PCM接口

为了方便操作访问，alsa-lib中封装了相关接口,通过pcmCXDXp/pcmCXDXc节点(/dev/s-nd/pcmCXDXx)去实现播放、录音功能。

主要涉及到的接口：

```
snd_pcm_open
snd_pcm_info
snd_pcm_hw_params_any
snd_pcm_hw_params_set_access
snd_pcm_hw_params_set_format
snd_pcm_hw_params_set_channels
snd_pcm_hw_params_set_rate_near
snd_pcm_hw_params_set_buffer_size_near
snd_pcm_hw_params
snd_pcm_sw_params_current
snd_pcm_sw_params
snd_pcm_readi
snd_pcm_writei
snd_pcm_close
```

详细pcm接口说明请查阅：

https://www.alsa-project.org/alsa-doc/alsa-lib/pcm.html

https://www.alsa-project.org/alsa-doc/alsa-lib/group___p_c_m.html

接口使用例子可以参考aplay,arecord的实现，代码可以在alsa-utils中找到(dl/alsa-utils-1.1.0.tar.bz2)