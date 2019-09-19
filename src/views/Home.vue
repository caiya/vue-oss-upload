<template>
  <div class="home">
    <input class="file" name="file" type="file" accept="image/png,image/gif,image/jpeg" @change="getFile($event)"/>
    <button  @click="submit($event)">开始上传</button>
    <div style="background-color: #ccc; margin-top: 20px;" v-show="!!fileMd5">
      <span>文件的md5：{{fileMd5}}</span>
    </div>
    <div style="background-color: #ccc; margin-top: 20px;" v-show="!!fileInfo">
      <span>上传返回信息：{{fileInfo}}</span>
    </div>
  </div>
</template>

<script>

import axios from 'axios'
import BMF from 'browser-md5-file'
const bmf = new BMF()

export default {
  name: 'home',
  data() {
    return {
      file: null,
      policy: {
        host: '',
        accessid: '',
        policy: '',
        signature: '',
        expire: '',
        callback: '',
        dir: ''
      },
      fileInfo: '',
      fileMd5: ''
    }
  },
  components: {

  },
  methods: {
    getFile(event) {
      this.file = event.target.files[0]
      bmf.md5(
        this.file, (err, md5) => {
          console.log('md5 string:', md5); // 97027eb624f85892c69c4bcec8ab0f11
          this.fileMd5 = md5
        }
      );
    },
    async submit(event) {
      event.preventDefault();
      const policyResult = await this.getUploadPolicy()
      this.policy = policyResult.data

      // 开始上传
      this.upload(data => {
        console.log(data)
        this.fileInfo = JSON.stringify(data)
      })
    },
    getUploadPolicy() {
      return axios.get('http://127.0.0.1:3000/users/getUploadPolicy')
    },
    getSuffix(filename) {
      let pos = filename.lastIndexOf('.')
      let suffix = ''
      if (pos != -1) {
          suffix = filename.substring(pos)
      }
      return suffix;
    },
    upload(cb) {
      let formdata = new FormData();
      formdata.append("key", this.policy.dir + this.fileMd5 + this.getSuffix(this.file.name));
      formdata.append("policy", this.policy.policy);
      formdata.append("OSSAccessKeyId", this.policy.accessid);
      formdata.append("success_action_status", '200');
      formdata.append("callback", this.policy.callback);
      formdata.append("signature", this.policy.signature);
      formdata.append("file", this.file);
      let xmlhttp = new XMLHttpRequest();
      xmlhttp.responseType = 'json'
      xmlhttp.open("POST", this.policy.host, true);
      xmlhttp.onreadystatechange = function() {
        if (xmlhttp.readyState === 4) {
          cb(xmlhttp.response)
        }
      }
      xmlhttp.send(formdata);
  }
  }
}
</script>
<style scoped>
</style>