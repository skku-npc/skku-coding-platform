<template>
  <div class="setting-main">
    <div class="section-title">
      {{ $t('m.Avatar_Setting') }}
    </div>
    <template v-if="!avatarOption.imgSrc">
      <Upload
        type="drag"
        class="mini-container"
        accept=".jpg,.jpeg,.png,.bmp,.gif"
        action=""
        :before-upload="handleSelectFile"
      >
        <div style="padding: 30px 0">
          <Icon
            type="ios-cloud-upload"
            size="52"
            style="color: #3399ff"
          />
          <p>Drop here, or click to select manually</p>
        </div>
      </Upload>
    </template>

    <template v-else>
      <div class="flex-container">
        <div class="cropper-main inline">
          <vueCropper
            ref="cropper"
            auto-crop
            fixed
            :auto-crop-width="200"
            :auto-crop-height="200"
            :img="avatarOption.imgSrc"
            :output-size="avatarOption.size"
            :output-type="avatarOption.outputType"
            :info="true"
            @realTime="realTime"
          />
        </div>
        <ButtonGroup
          vertical
          class="cropper-btn"
        >
          <Button @click="rotate('left')">
            <Icon
              type="arrow-return-left"
              size="20"
            />
          </Button>
          <Button @click="rotate('right')">
            <Icon
              type="arrow-return-right"
              size="20"
            />
          </Button>
          <Button @click="reselect">
            <Icon
              type="refresh"
              size="20"
            />
          </Button>
          <Button @click="finishCrop">
            <Icon
              type="checkmark-round"
              size="20"
            />
          </Button>
        </ButtonGroup>
        <div
          class="cropper-preview"
          :style="previewStyle"
        >
          <div :style=" preview.div">
            <img
              :src="avatarOption.imgSrc"
              :style="preview.img"
            >
          </div>
        </div>
      </div>
    </template>
    <Modal
      v-model="uploadModalVisible"
      title="Upload the avatar"
    >
      <div class="upload-modal">
        <p class="notice">
          Your avatar will be set to:
        </p>
        <img :src="uploadImgSrc">
      </div>
      <div slot="footer">
        <Button
          :loading="loadingUploadBtn"
          @click="uploadAvatar"
        >
          upload
        </Button>
      </div>
    </Modal>

    <div class="section-title">
      {{ $t('m.Profile_Setting') }}
    </div>
    <Form
      ref="formProfile"
      :model="formProfile"
    >
      <Row
        type="flex"
        :gutter="30"
        justify="space-around"
      >
        <Col :span="11">
        <FormItem label="Real Name">
          <Input v-model="formProfile.real_name" />
        </FormItem>
        <FormItem label="Default Programming Language">
          <Select v-model="formProfile.language">
            <Option
              v-for="lang in languages"
              :key="lang.value"
              :value="lang.value"
            >
              {{ lang.value }}
            </Option>
          </Select>
        </FormItem>
        <Form-item>
          <Button
            type="primary"
            :loading="loadingSaveBtn"
            @click="updateProfile"
          >
            Save All
          </Button>
        </Form-item>
        </Col>

        <Col :span="11">
        <Form-item label="Mood">
          <Input v-model="formProfile.mood" />
        </Form-item>
        <Form-item label="Blog">
          <Input v-model="formProfile.blog" />
        </Form-item>
        <Form-item label="Github">
          <Input v-model="formProfile.github" />
        </Form-item>
        </Col>
      </Row>
    </Form>
  </div>
</template>

<script>
import api from '@oj/api'
import utils from '@/utils/utils'
import { VueCropper } from 'vue-cropper'
import { types } from '@/store'

export default {
  components: {
    VueCropper
  },
  data () {
    return {
      loadingSaveBtn: false,
      loadingUploadBtn: false,
      uploadModalVisible: false,
      preview: {},
      uploadImgSrc: '',
      avatarOption: {
        imgSrc: '',
        size: 0.8,
        outputType: 'png'
      },
      languages: [
        { value: 'C' },
        { value: 'C++' },
        { value: 'Golang' },
        { value: 'Java' },
        { value: 'Python2' },
        { value: 'Python3' }
      ],
      formProfile: {
        real_name: '',
        mood: '',
        blog: '',
        github: '',
        language: ''
      }
    }
  },
  computed: {
    previewStyle () {
      return {
        width: this.preview.w + 'px',
        height: this.preview.h + 'px',
        overflow: 'hidden'
      }
    }
  },
  mounted () {
    const profile = this.$store.state.user.profile
    Object.keys(this.formProfile).forEach(element => {
      if (profile[element] !== undefined) {
        this.formProfile[element] = profile[element]
      }
    })
  },
  methods: {
    checkFileType (file) {
      if (!/\.(gif|jpg|jpeg|png|bmp|GIF|JPG|PNG)$/.test(file.name)) {
        this.$Notice.warning({
          title: 'File type not support',
          desc: 'The format of ' + file.name + ' is incorrect ，please choose image only.'
        })
        return false
      }
      return true
    },
    checkFileSize (file) {
      // max size is 2MB
      if (file.size > 2 * 1024 * 1024) {
        this.$Notice.warning({
          title: 'Exceed max size limit',
          desc: 'File ' + file.name + ' is too big, you can upload a image up to 2MB in size'
        })
        return false
      }
      return true
    },
    handleSelectFile (file) {
      const isOk = this.checkFileType(file) && this.checkFileSize(file)
      if (!isOk) {
        return false
      }
      const reader = new window.FileReader()
      reader.onload = (e) => {
        this.avatarOption.imgSrc = e.target.result
      }
      reader.readAsDataURL(file)
      return false
    },
    realTime (data) {
      this.preview = data
    },
    rotate (direction) {
      if (direction === 'left') {
        this.$refs.cropper.rotateLeft()
      } else {
        this.$refs.cropper.rotateRight()
      }
    },
    reselect () {
      this.$Modal.confirm({
        content: 'Are you sure to disgard the changes?',
        onOk: () => {
          this.avatarOption.imgSrc = ''
        }
      })
    },
    finishCrop () {
      this.$refs.cropper.getCropData(data => {
        this.uploadImgSrc = data
        this.uploadModalVisible = true
      })
    },
    uploadAvatar () {
      this.$refs.cropper.getCropBlob(blob => {
        const form = new window.FormData()
        const file = new window.File([blob], 'avatar.' + this.avatarOption.outputType)
        form.append('image', file)
        this.loadingUploadBtn = true
        this.$http({
          method: 'post',
          url: 'upload_avatar',
          data: form,
          headers: { 'content-type': 'multipart/form-data' }
        }).then(res => {
          this.loadingUploadBtn = false
          this.$success('Successfully set new avatar')
          this.uploadModalVisible = false
          this.avatarOption.imgSrc = ''
          this.$store.dispatch('getProfile')
        }, () => {
          this.loadingUploadBtn = false
        })
      })
    },
    updateProfile () {
      this.loadingSaveBtn = true
      const updateData = utils.filterEmptyValue(Object.assign({}, this.formProfile))
      api.updateProfile(updateData).then(res => {
        this.$success('Success')
        this.$store.commit(types.CHANGE_PROFILE, { profile: res.data.data })
        this.loadingSaveBtn = false
      }, _ => {
        this.loadingSaveBtn = false
      })
    }
  }
}
</script>

<style lang="less" scoped>
  .inline {
    display: inline-block;
  }

  .copper-img {
    width: 400px;
    height: 300px;
  }

  .flex-container {
    flex-wrap: wrap;
    justify-content: flex-start;
    margin-bottom: 10px;
    .cropper-main {
      flex: none;
      .copper-img;
    }
    .cropper-btn {
      flex: none;
      vertical-align: top;
    }
    .cropper-preview {
      flex: none;
      /*margin: 10px;*/
      margin-left: 20px;
      box-shadow: 0 0 1px 0;
      .copper-img;
    }
  }

  .upload-modal {
    .notice {
      font-size: 16px;
      display: inline-block;
      vertical-align: top;
      padding: 10px;
      padding-right: 15px;
    }
    img {
      box-shadow: 0 0 1px 0;
      border-radius: 50%;
    }
  }
</style>
