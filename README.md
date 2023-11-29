# Bert-VITS2 Cook Book

_这不是Bert-VITS2作者的官方教程，源自我使用过程的随手记录，不对正确性做保证。_

_这是2023年9月写下的教程，其中部分内容可能已经过时，仅供参考。如果在使用过程中发现教程有问题，欢迎讨论（虽然我也不一定能帮你解决就是了）或者pull request进行修正。_

## 1. 创建虚拟环境

```shell
virtualenv venv
source venv/bin/activate
```

## 2. 安装依赖项

```shell
pip install -r requirements.txt
```

## 3. 准备数据

1. 自定义数据集：

   * 若干条语音`path/to/dataset/<speaker_name>/*.wav`，可以放在任意路径下，因为会在`filelists/genshin.list`中注明；注意，语音文件的采样率必须与配置文件`configs/config.json`中的`data.sampling_rate`一致，如果不一致可以使用脚本`resample.py`进行重采样

   * 语音内容标注`filelist/genshin.list`（这个文件名是在脚本`preprocess_text.py`中写死的），其中每行的格式为：

     `{wav_path}|{speaker_name}|{language}|{text}`

     这是一个具体的例子：

     `/root/my_dataset/diona/114514.wav|diona|ZH|哼哼，快快开始激动人心的新人对局吧！`

2. 执行脚本`preprocess_text.py`对`filelist/genshin.list`做预处理：

   ```shell
   python preprocess_text.py
   ```

   这一步会在目录`filelists`生成中间文件`genshin.list.cleaned`以及用于模型训练评估的`train.list`和`val.list`，同时会根据数据集中实际的角色修改`configs/config.json`中的`spk2id`字段

## 4. 准备底模

下载底模：[Stardust_minus/Bert-VITS2 - Bert-VITS2 - OpenI - 启智AI开源社区提供普惠算力！ (pcl.ac.cn)](https://openi.pcl.ac.cn/Stardust_minus/Bert-VITS2/modelmanage/model_filelist_tmpl?name=Bert-VITS2底模)，在`logs`下新建目录`<speaker_name>`将下载好的模型权重文件放入其中

## 5. 下载BERT模型权重

在本repo中有目录`bert/chinese-roberta-wwm-ext-large`，这一目录对应的原始repo为[hfl/chinese-roberta-wwm-ext-large at main (huggingface.co)](https://huggingface.co/hfl/chinese-roberta-wwm-ext-large/tree/main)，可以完整克隆进行替换，也可以下载`pytorch_model.bin`放入目录`bert/chinese-roberta-wwm-ext-large`

## 6. 使用BERT生成prosody embedding

```shell
python bert_gen.py
```

这一步会在目录`path/to/dataset/<speaker_name>`中为每个音频生成`*.bert.pt`

## 7. 开始训练

```shell
python train_ms.py -c configs/config.json -m <speaker_name>
```

其中`<speaker_name>`为第四步中保存底模的目录名。训练过程中默认每1000步保存一次checkpoint，保存路径为`logs/<speaker_name>/*.pth`

## 8. Web UI，启动！

```shell
python webui.py --model logs/<speaker_name>/G_xxx.pth
```

## References

* 原始Repo: [Stardust-minus/Bert-VITS2: vits2 backbone with bert (github.com)](https://github.com/Stardust-minus/Bert-VITS2)
* [YYuX-1145/Bert-VITS2-Integration-package: vits2 backbone with bert (github.com)](https://github.com/YYuX-1145/Bert-VITS2-Integration-package/tree/main)
* [Bert-Vits2 手把手本地部署录屏教程_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV18N4y1Q7JK/?spm_id_from=333.999.0.0)
