<template>
  <div>
    <spaceLayout>
      <div class="upload-single-container">
        <router-link :to="{ name: 'Upload' }" class="back-link">
          <v-icon color="#E56D9B">mdi-chevron-left</v-icon>
          Back to Selection
        </router-link>
        <div class="container">
          <div class="cover-title side-title">Single Cover</div>
          <img-upload
          :img-upload-done="imgUploadDone"
          :update-type="'single'"
          class="app-icon"
          @doneImageUpload="doneImageUpload"
          >
            <div
              slot="uploadButton"
              class="user-avatar"
            >
              <div class="edit">
                <v-icon color="#FFF">mdi-camera</v-icon>
                Single Cover
              </div>
              <img
                id="avatar"
                v-if="singleCover"
                slot="description"
                :src="singleCover"
                alt="avatar"
              >
              <img v-else id="new-logo" src="../../assets/image/single.png" style="margin-top: 10px;"/>
            </div>
          </img-upload>
          <div class="name-title side-title">Music Name</div>
          <v-text-field
            v-model="singleTitle"
            label="Solo"
            placeholder="Enter Your Music Title..."
            solo
            color="#FFF"
            style="margin-top: 16px;"
          ></v-text-field>
          <div class="name-desp side-title">Description</div>
          <v-textarea
            v-model="singleDesp"
            solo
            name="input-7-4"
            label="Your Single Description..."
          ></v-textarea>
          <div class="name-desp side-title">Demo Duration</div>
          <v-select
            v-model="duration"
            :items="items"
            label="Demo duration"
            solo
          ></v-select>
          <v-file-input
            v-model="file"
            color="#FFF"
            chips
            placeholder="Select your file"
            prepend-icon="mdi-paperclip"
            outlined
            accept="audio/mp3,audio/flac,audio/wave,audio/wav,audio/ogg,audio/mpeg"
            :show-size="1000"
            style="margin-top: 16px;"
          >
            <template v-slot:selection="{ index, text }">
              <v-chip
                v-if="index < 2"
                color="deep-purple accent-4"
                dark
                label
                small
              >
                {{ text }}
              </v-chip>

              <span
                v-else-if="index === 2"
                class="overline grey--text text--darken-3 mx-2"
              >
                +{{ files.length - 2 }} File(s)
              </span>
            </template>
          </v-file-input>
          <v-btn color="#E56D9B" depressed light class="side-title" :loading="submitBtnLoading" @click="submit">Submit</v-btn>
        </div>
      </div>
      <v-snackbar
        v-model="snackbar"
        color="#00C853"
        timeout="3000"
        top="top"
      >
        Image Read Successful

        <template v-slot:action="{ attrs }">
          <v-btn
            dark
            text
            v-bind="attrs"
            @click="snackbar = false"
          >
            Close
          </v-btn>
        </template>
      </v-snackbar>
      <v-snackbar
        v-model="singleSnackbar"
        color="#00C853"
        timeout="3000"
        top="top"
      >
        Single Release Successful

        <template v-slot:action="{ attrs }">
          <v-btn
            dark
            text
            v-bind="attrs"
            @click="singleSnackbar = false"
          >
            Close
          </v-btn>
        </template>
      </v-snackbar>
      <v-snackbar
        v-model="failSnackbar"
        color="#E53935"
        timeout="3000"
        top="top"
        style="margin-top: 16px;"
      >
        {{ failMessage }}

        <template v-slot:action="{ attrs }">
          <v-btn
            dark
            text
            v-bind="attrs"
            @click="failSnackbar = false"
          >
            Close
          </v-btn>
        </template>
      </v-snackbar>
    </spaceLayout>
  </div>
</template>

<script>

import imgUpload from '@/components/imgUpload/imgUpload.vue'
import spaceLayout from '@/components/Layout/Space.vue'

import singleDefault from '@/assets/image/single.png'
import { mapActions, mapState } from 'vuex'

export default {
  components: {
    imgUpload,
    spaceLayout
  },
  data () {
    return {
      file: null,
      duration: '',
      singleDefault: singleDefault,
      singleTitle: '',
      singleDesp: '',
      needUpload: true,
      attachments: [],
      singleCover: '',
      fileContent: '',
      fileRaw: '',
      fileName: '',
      imgUploadDone: 0,
      music: '',
      musicContent: '',
      snackbar: false,
      failSnackbar: false,
      singleSnackbar: false,
      failMessage: '',
      submitBtnLoading: false,
      items: ['10s', '30s', '60s', 'Off']
    }
  },
  computed: {
    ...mapState(['singleCoverFile', 'isLoggedIn', 'keyFileContent', 'singleUploadComplete', 'singleLink'])
  },
  watch: {
    singleUploadComplete (val) {
      this.submitBtnLoading = false
      this.singleSnackbar = true

      setTimeout(() => {
        this.$router.push({ path: '/music/' + this.singleLink })
      }, 2000)
    }
  },
  methods: {
    ...mapActions(['uploadSingleCoverFile', 'uploadSingle']),
    submit () {
      this.submitBtnLoading = true
      if (this.singleCover === '') {
        this.failMessage = 'A cover for a single release is required'
        this.failSnackbar = true
        return
      }

      if (this.singleTitle === '') {
        this.failMessage = 'A title for a single release is required'
        this.failSnackbar = true
        return
      }

      if (this.singleDesp === '') {
        this.failMessage = 'A description for a single release is required'
        this.failSnackbar = true
        return
      }

      if (!this.duration) {
        this.failMessage = 'The demo duration is required'
        this.failSnackbar = true
        return
      }

      if (!this.file) {
        this.failMessage = 'A source music file for a single release is required'
        this.failSnackbar = true
        return
      }

      let imgType = {
        png: 'image/png',
        jpeg: 'image/jpeg',
        jpg: 'image/jpeg',
        webp: 'image/webp'
      }
      let ext = this.singleCoverFile.name.split('.').pop()
      console.log('Content-Type:', imgType[ext])

      let audioType = {
        mp3: 'audio/mp3',
        flac: 'audio/flac',
        wav: 'audio/wav',
        ogg: 'audio/ogg'
      }

      let aext = this.file.name.split('.').pop()
      console.log('Content-Type:', audioType[aext])
      const reader = new FileReader()
      reader.readAsArrayBuffer(this.file)
      reader.onload = async (e) => {
        this.music = e.target.result
        this.musicContent = this.file

        this.uploadSingle({
          img: { data: this.fileRaw, type: imgType[ext] },
          music: { data: this.music, type: audioType[aext] },
          key: this.keyFileContent,
          single: {
            title: this.singleTitle,
            desp: this.singleDesp
          }
        })
      }
    },
    doneImageUpload () {
      this.singleCover = this.singleCoverFile
      this.fileName = this.singleCover.name
      const reader = new FileReader()
      reader.readAsDataURL(this.singleCover)
      reader.onload = async (e) => {
        try {
          this.fileContent = e.target.result
          this.fileRaw = this.fileContent
          this.needUpload = false
          this.singleCover = this.fileRaw
          this.snackbar = true
        } catch (err) {
          this.imgFailSnackbar = true
        }
      }
      this.imgUploadDone += Date.now()
      this.snackbar = true
    }
  },
  mounted () {
    document.title = 'Upload a new Single - ArcLight'

    setTimeout(() => {
      if (!this.isLoggedIn) {
        this.failMessage = 'Login is required to upload'
        this.failSnackbar = true

        setTimeout(() => {
          this.$router.push({ name: 'Landing' })
        }, 3000)
      }
    }, 3000)
  }
}
</script>

<style lang="less" scoped>
.upload-single-container {
  margin: 20px auto;
  max-width: 840px;
  width: 100%;
  padding: 0 20px;
}

.back-link {
  display: block;
  text-align: left;
  margin-bottom: 16px;
}

.side-title {
  font-weight: 600;
  display: block;
  text-align: left;
  color: white;
  margin-bottom: 16px;
}

.user-avatar {
  margin: 0 auto;
}

.app-icon {
  cursor: pointer;
  margin-bottom: 16px;
  display: flex;
  align-items: center;
  flex-direction: column;
  justify-content: center;
  width:150px;
  height:150px;
  object-fit: cover;
  background:rgba(90, 90, 90, 0.247);
  border-radius:8px;
  &:hover {
    background-color: rgb(80, 80, 80);
  }
  .user-avatar > img {
    width: 130px;
    height: 130px;
  }
  &:hover .edit {
    display: flex;
  }
  .edit {
    width: 130px;
    height: 130px;
    margin-top: 10px;
    margin-left: 10px;
    display: none;
    cursor: pointer;
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    color: #eee;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    background: rgba(0, 0, 0, 0.6);
    font-size: 14px;
    .el-icon-camera {
      font-size: 24px;
      margin-bottom: 4px;
      color: #eee;
    }
  }
}

#avatar {
  margin-top: 10px;
}

/deep/ .v-text-field__slot > input {
  color: white;
  &::placeholder { /* Chrome, Firefox, Opera, Safari 10.1+ */
    color: white;
    opacity: 1; /* Firefox */
  }
}

/deep/ .v-text-field__slot > textarea {
  color: white;
}

/deep/ .v-text-field__slot > label {
  color: white;
}

/deep/ .v-icon--link {
  color: white;
}

/deep/ .v-select__selection {
  color: white !important;
}

/deep/ .v-select__slot > label {
  color: white;
}

/deep/ .v-input__icon > i {
  color: white;
}

/deep/ .v-text-field--is-booted {
  color: #E56D9B;
  border-color: white;
}

/deep/ .v-file-input__text.v-file-input__text--placeholder {
  color: white;
}

/deep/ .theme--light.v-text-field--solo>.v-input__control>.v-input__slot {
  background-color: rgba(51,51,51,0.8);
}
</style>