---
layout: post
title: "解决Android录音iOS兼容性"
date: 2014-09-11 10:50:25
categories: Android开发

---

最近在开发过程中遇到问题，Android客户端录制的AMR_NB编码的文件在iOS设备商播放失败。  
经过反复百度谷歌，得知iOS需要借助AmrToWave的转换，这无疑给iOS开发者带来不便。随即想到更改录音时的参数，经过测试，使用AAC可以解决该问题，而且Android本身使用MediaPlayer类播放即可，减轻了iOS开发者的工作量。录音示例部分代码如下：  


    MediaRecoder mediaRecorder = new MediaRecorder();  
	mediaRecorder.setAudioSource(MediaRecorder.AudioSource.MIC);  
	mediaRecorder.setOutputFormat(MediaRecorder.OutputFormat.THREE_GPP);  
	mediaRecorder.setOutputFile(APPCFG.VOICE_FILE);  
	mediaRecorder.setAudioEncoder(MediaRecorder.AudioEncoder.AAC);  