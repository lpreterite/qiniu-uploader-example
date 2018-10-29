<template>
<div>
    <p>
        <label>七牛token：<input type="text" v-model="token" style="width: 400px"></label>
    </p>
    <div>
        <div>
            <button type="button" @click="uploader.openFinder(onFileChanged, {multiple: true})">选择文件</button>
        </div>
        <ul class="progress">
            <li v-for="(progress, index) in fileset" :key="index">
                <div class="progress__title">
                    {{ progress.blob.name }}<small>（{{ progress.fileKey }}）</small>
                    <span>
                        <button type="button" v-if="!progress.uploading && progress.stage !== 'cancel'" @click="upload(progress.fileKey)">上传</button>
                        <button type="button" v-if="progress.blockCount>1 && progress.uploading" @click="uploader.cancel(progress.fileKey)">暂停</button>
                        <button type="button" v-if="progress.blockCount>1 && progress.stage === 'cancel'" @click="upload(progress.fileKey)">继续</button>
                    </span>
                </div>
                <div class="progress__meta" v-if="progress.uploading">上传进度：{{progress.value}}% - {{(progress.loaded/1024/1024).toFixed(2)}}mb / {{(progress.total/1024/1024).toFixed(2)}}mb —— 当前块：{{progress.blockIndex}} 当前阶段：{{progress.stage}}</div>
                <div class="progress__meta" v-if="!progress.uploading && progress.errors.length > 0">{{ progress.errors[0].message }}</div>
            </li>
        </ul>
    </div>
</div>
</template>

<script>
import uploader from '@packy-tang/qiniu-uploader';
export default {
    data(){
        return {
            uploader,
            token: "fqW-wERKESARsZ9_8-sZCvRhLNqNOIgC3ijJsG-9:-mLuebsjx2-GC7GSDXax39hNOq4=:eyJzY29wZSI6Imhla2F0ZSIsImRlYWRsaW5lIjoxNTM5ODUzMzI5fQ==",
            loading: false,
            fileset: []
        }
    },
    mounted(){
        uploader.init({
            onChanged: ()=>{
                this.fileset = uploader.getAllFile()
            }
        })
    },
    methods: {
        openFinder(fn){
            return uploader.openFinder(fn)
        },
        onFileChanged(files, uploader){
            Array.prototype.forEach.call(files, async (blob, index)=>{
                const fileKey = await uploader.getFileKey(blob)
                uploader.addFile(fileKey, blob)
            })
        },
        upload(fileKey){
            this.uploader.upload(fileKey, { token:this.token, url: "//up-z2.qiniup.com" })
        },
        cancel(key){
            uploader.cancel(key)
        }
    }
}
</script>

<style>
.progress{
    text-align: left;
    width: 80%;
    margin: 35px auto;
    list-style: none;
}
.progress__title{
    margin: 10px 0;
}
.progress__title small{
    color: #999;
}
.progress__meta{
    font-size: small;
    padding-left: 1em;
    margin: 10px 0 20px;
}
.progress__meta:before{
    content: "-";
    margin-right: 1em;
}
</style>
