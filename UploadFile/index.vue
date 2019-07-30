<template>
  <div>
    <el-upload
      :action="uploadAction"
      class="upload-demo"
      drag
      :on-error="handleUploadError"
      :on-success="handleUploadSuccess"
      name="file"
      :file-list="fileList"
      :data="uploadParams"
      :on-remove="handleRemove"
      :before-upload="beforeAvatarUpload"
      :multiple="isMultiple"
      :on-preview="handlePreview">
      <i class="el-icon-upload"></i>
      <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
      <div class="el-upload__tip" slot="tip">{{tips}}</div>
    </el-upload>
  </div>
</template>
<script>
  import { mapGetters } from 'vuex'
  import _ from 'lodash'
  // import params from '../../conf/params'
  import commonApi from '@/api/common'
  export default {
    props: {
      maxCount: {},
      fileList: {
        type: Array,
        default () {
          return []
        }
      },
      isUploadToTemp: {
        type: Boolean,
        default: true
      },
      isMultiple: {
        type: Boolean,
        default: false
      },
      tips: {
        type: String,
        default: '支持png，jpg，mp4，word，excel，ppt，pdf，rar，zip等'
      },
      allowedTypes: {
        type: Array,
        default () {
          return []
        }
      }
    },
    data () {
      return {
        fileCount: 0,
        uploadParams: {
        },
        uploadAction: ''
      }
    },
    computed: {
      ...mapGetters([
        'token'
      ])
    },
    methods: {
      getOssUploadConfig (file, resolve, reject) {
        commonApi.getOssUploadConfig({
          type: 'file',
          temp: this.isUploadToTemp ? 1 : 0,
          suffix: this.get_suffix(file.name)
        }).then(response => {
          const res = response.body.data
          this.uploadAction = res.host
          const fileName = res.file_dir + new Date().getTime() + '_' + this.random_string(5) + '.' + this.get_suffix(file.name)
          file.uploadKey = fileName
          this.uploadParams = {
            key: fileName,
            policy: res.policy,
            OSSAccessKeyId: res.access_id,
            success_action_status: '200',
            signature: res.signature
          }
          resolve()
        }, response => {
          reject()
          this.$message.error(response.body.message)
        })
      },
      beforeAvatarUpload (file) {
        this.fileCount++
        if (this.fileCount > this.maxCount) {
          this.fileCount--
          this.$message.warning(`文件数量超过${this.maxCount}个`)
          return false
        }
        const self = this
        const nameArr = file.name && file.name.split('.')
        if (nameArr && nameArr[nameArr.length - 1] === 'rar') {
          file.fileType = 'rar'
        }
        if (this.allowedTypes.length) {
          if (this.allowedTypes.indexOf(file.type) === -1 && this.allowedTypes.indexOf(file.fileType) === -1) {
            this.$message.error('不允许上传的文件类型')
            return false
          }
        }
        return new Promise((resolve, reject) => {
          self.getOssUploadConfig(file, resolve, reject)
        })
      },
      get_suffix (filename) {
        const pos = filename.lastIndexOf('.')
        let suffix = ''
        if (pos !== -1) {
          suffix = filename.substring(pos + 1)
        }
        return suffix
      },
      random_string (len) {
        len = len || 32
        const chars = 'ABCDEFGHJKMNPQRSTWXYZabcdefhijkmnprstwxyz2345678'
        const maxPos = chars.length
        let pwd = ''
        for (let i = 0; i < len; i++) {
          pwd += chars.charAt(Math.floor(Math.random() * maxPos))
        }
        return pwd
      },
      handleUploadError () {
        this.$message.error('文件上传失败，请检测文件大小或类型是否合格')
      },
      handleUploadSuccess (response, file, fileList) {
        if (!this.isMultiple) {
          _.remove(fileList, obj => {
            return obj.uid !== file.uid
          })
        }
        commonApi.createFile(
          {
            driver: 'oss',
            path: file.raw.uploadKey,
            file_name: file.name,
            type: 'file'
          }
        ).then(response => {
          file.response = response.body
          this.$emit('upload-success', response.body)
        }, response => {
          this.fileCount--
          this.$message.error(response.body.message)
          _.remove(fileList, obj => {
            return obj.uid === file.uid
          })
        })
      },
      handleRemove (file) {
        if (file.id) {
          this.$emit('remove-file', file)
        }
        if (file.response) {
          this.fileCount--
          this.$emit('remove-file', file.response)
        }
      },
      handlePreview (file) {
        file.id && window.open('/file/download/' + file.id)
        file.response && window.open('/file/download/' + file.response.data.id)
      }
    }
  }
</script>
<style scoped>
</style>
