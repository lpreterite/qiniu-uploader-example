<template>
  <div>
    <button @click="onClick">Upload</button>
  </div>
</template>

<script>
import { UploadManager, getFileKey } from '@packy-tang/qiniu-uploader';

const blockSize = 1<<22;
const uploadManager = new UploadManager({ blockSize });
const token = "fqW-wERKESARsZ9_8-sZCvRhLNqNOIgC3ijJsG-9:RYw9feTkgfJZIVmh3dflUCV6xbo=:eyJzY29wZSI6Imhla2F0ZSIsImRlYWRsaW5lIjoxNTM5NTg2NjM0fQ=="

export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  methods: {
    onClick(){
      return this.openFinder({})
    },
    openFinder({ accept = '*' }) {
        const fileinput = document.createElement('input');
        fileinput.type = 'file';
        fileinput.accept = accept;
        fileinput.style.display = "none";
        fileinput.addEventListener('change', e => {
            this._fileChanged(e);
        }, false);
        return fileinput.click();
    },
    _fileChanged(e){
      this.upload(e.target.files[0])
    },
    async upload(file){
      const fileKey = await getFileKey(file, blockSize);
      const uploader = uploadManager.getUploader(fileKey, file, token);
      uploader.onprogress((uploader)=>{
          const total = Math.floor((uploader.status.total || 0) / 1024);
          const uploaded = Math.floor((uploader.status.uploaded || 0) / 1024);
          const blockIndex = uploader.status.block.index;
          console.log(`上传进度：${uploader.progress.value}% - ${uploaded}kb / ${total}kb —— 当前块：${blockIndex} 当前阶段：${uploader.progress.stage}`);
      });
      uploader
          .action()
          .then(()=>{
              console.log('上传成功');
          })
          .catch(e=>{
              if(e.message.indexOf('000')) console.error('上传出现意外错误');
              else if(e.message.indexOf('401')) console.error('未授权或授权过期，请检测token');
              else console.error(e.message);
          });
    }
  }
}
</script>