<template>
    <div>
        <button @click="openFinder(onFileChanged)">Upload</button>
        <button @click="cancel">Cancel</button>
    </div>
</template>

<script>
import uploader from '@packy-tang/qiniu-uploader';
const token = "fqW-wERKESARsZ9_8-sZCvRhLNqNOIgC3ijJsG-9:Bd3jJ9JgcOIpGhJHJDi6RHHdIic=:eyJzY29wZSI6Imhla2F0ZSIsImRlYWRsaW5lIjoxNTM5Njg1Nzc3fQ=="

export default {
    methods: {
        openFinder(fn){
            return uploader.openFinder(fn)
        },
        async onFileChanged(files, uploader){
            console.time('GetFileKey')
            const fileKey = await uploader.getFileKey(files[0])
            console.timeEnd('GetFileKey')

            uploader.upload(fileKey, files[0], {
                url: '//up-z2.qiniup.com',
                token,
                onUploadProgress: ({fileKey, loaded, total, progressStage, progressValue, blockIndex})=>{
                    total = Math.floor((total || 0) / 1024);
                    loaded = Math.floor((loaded || 0) / 1024);
                    console.log(`上传进度：${progressValue}% - ${loaded}kb / ${total}kb —— 当前块：${blockIndex} 当前阶段：${progressStage}`);
                },
                onFail:(errors, {isCancel})=>{
                    if(isCancel) console.log(errors)
                }
            })
        },
        cancel(){
            uploader.cancel()
        }
    }
}
</script>