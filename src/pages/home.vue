<template>
  <div class="container">
    <el-collapse @change="init()">
      <el-collapse-item title="配置信息" name="aliyun">
        <el-form ref="ruleFormRef" :model="ruleForm" :rules="rules" label-width="150px" status-icon>
          <el-form-item label="AccessKeyId" prop="accessKeyId">
            <el-input v-model="ruleForm.accessKeyId" />
          </el-form-item>
          <el-form-item label="AccessKeySecret" prop="accessKeySecret">
            <el-input v-model="ruleForm.accessKeySecret" />
          </el-form-item>
          <el-form-item label="Bucket" prop="bucket">
            <el-input v-model="ruleForm.bucket" />
          </el-form-item>
          <el-form-item label="Region" prop="region">
            <el-input v-model="ruleForm.region" />
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="updateForm()">更新配置</el-button>
          </el-form-item>
        </el-form>
      </el-collapse-item>
    </el-collapse>
    <el-upload class="upload-container" drag :http-request="uploadRequest" :show-file-list="false">
      <el-icon class="el-icon--upload">
        <UploadFilled />
      </el-icon>
      <div class="el-upload__text">
        将文件拖到此处或 <em>点击上传</em>
      </div>
      <template #tip>
        <div class="el-upload__tip">
          PS：请先设置配置信息后再进行上传文件操作；
        </div>
      </template>
    </el-upload>
    <ul class="filelist el-upload-list el-upload-list--text">
      <li class="el-upload-list__item is-success" tabindex="0" v-for="item in fileList.value">
        <div class="el-upload-list__item-info" @click="copyUrl(item)"><a class="el-upload-list__item-name"><i
              class="el-icon el-icon--document"><svg viewBox="0 0 1024 1024" xmlns="http://www.w3.org/2000/svg">
                <path fill="currentColor"
                  d="M832 384H576V128H192v768h640V384zm-26.496-64L640 154.496V320h165.504zM160 64h480l256 256v608a32 32 0 0 1-32 32H160a32 32 0 0 1-32-32V96a32 32 0 0 1 32-32zm160 448h384v64H320v-64zm0-192h160v64H320v-64zm0 384h384v64H320v-64z">
                </path>
              </svg></i><span class="el-upload-list__item-file-name">{{ item.name }}</span></a></div>
        <label class="el-upload-list__item-status-label">
          <i class="el-icon el-icon--upload-success el-icon--circle-check">
            <svg viewBox="0 0 1024 1024" xmlns="http://www.w3.org/2000/svg">
              <path fill="currentColor"
                d="M512 896a384 384 0 1 0 0-768 384 384 0 0 0 0 768zm0 64a448 448 0 1 1 0-896 448 448 0 0 1 0 896z">
              </path>
              <path fill="currentColor"
                d="M745.344 361.344a32 32 0 0 1 45.312 45.312l-288 288a32 32 0 0 1-45.312 0l-160-160a32 32 0 1 1 45.312-45.312L480 626.752l265.344-265.408z">
              </path>
            </svg>
          </i>
        </label>
        <i class="el-icon el-icon--close" @click="delUrl(item)">
          <svg viewBox="0 0 1024 1024" xmlns="http://www.w3.org/2000/svg">
            <path fill="currentColor"
              d="M764.288 214.592 512 466.88 259.712 214.592a31.936 31.936 0 0 0-45.12 45.12L466.752 512 214.528 764.224a31.936 31.936 0 1 0 45.12 45.184L512 557.184l252.288 252.288a31.936 31.936 0 0 0 45.12-45.12L557.12 512.064l252.288-252.352a31.936 31.936 0 1 0-45.12-45.184z">
            </path>
          </svg>
        </i>
      </li>
    </ul>
  </div>
</template>

<style lang="scss" scoped>
.container {
  height: 100%;
  padding: 20px;
}

.upload-container {
  margin-top: 10px;
}

.filelist {
  height: 180px;
  overflow-y: auto;
}

.filelist::-webkit-scrollbar {
  /*滚动条整体样式*/
  width: 2px;
  /*高宽分别对应横竖滚动条的尺寸*/
  height: 1px;
}

.filelist::-webkit-scrollbar-thumb {
  /*滚动条里面小方块*/
  border-radius: 10px;
  -webkit-box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
  background: #535353;
}

.filelist::-webkit-scrollbar-track {
  /*滚动条里面轨道*/
  -webkit-box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
  border-radius: 10px;
  background: #EDEDED;
}
</style>

<script setup lang="ts">
import { onMounted, reactive, ref } from "vue";
import { ElMessage, ElMessageBox, ElButton, ElUpload, ElIcon, ElCollapse, ElCollapseItem, ElForm, ElFormItem, ElInput, FormInstance, UploadUserFile, UploadRequestOptions } from "element-plus";
import { UploadFilled } from "@element-plus/icons-vue";
import OSS from "ali-oss";

let client: OSS;

const ruleFormRef = ref<FormInstance>();

const ruleForm = reactive({
  accessKeyId: '',
  accessKeySecret: '',
  bucket: '',
  region: '',
});

const rules = reactive({
  accessKeyId: [
    { required: true, message: 'Please input AccessKeyId', trigger: 'blur' },
  ],
  accessKeySecret: [
    { required: true, message: 'Please input AccessKeySecret', trigger: 'blur' },
  ],
  bucket: [
    { required: true, message: 'Please input Bucket', trigger: 'blur' },
  ],
  region: [
    { required: true, message: 'Please input Region', trigger: 'blur' },
  ],
});

const fileList = reactive<{ value: OSS.ObjectMeta[] }>({
  value: [],
})

const open = (message: string) => {
  ElMessage(message)
}

const refresh = async () => {
  const result = await client.listV2({}, {});
  fileList.value = result.objects.reverse();
}

const copyUrl = (value: OSS.ObjectMeta) => {
  window.utools.copyText(value.url);
  open('链接复制成功~');
}

const delUrl = async (value: OSS.ObjectMeta) => {
  ElMessageBox.confirm(
    '确认删除此文件吗?',
    'Warning',
    {
      confirmButtonText: '确认',
      cancelButtonText: '取消',
      type: 'warning',
    }
  )
    .then(async () => {
      await client.delete(value.name);
      await refresh();
      open('文件删除成功~');
    })
}

const init = async () => {
  Object.keys(ruleForm).forEach((key) => {
    const value = utools.dbStorage.getItem(key);
    if (value) {
      Reflect.set(ruleForm, key, value)
    }
  })
  const result = Object.keys(ruleForm).filter(key => Reflect.get(ruleForm, key));
  if (result.length === 4) {
    client = new OSS({
      region: ruleForm.region,
      accessKeyId: ruleForm.accessKeyId,
      accessKeySecret: ruleForm.accessKeySecret,
      bucket: ruleForm.bucket,
    });
    await refresh();
  }
}

const updateForm = async () => {
  const formRef = ruleFormRef.value;
  await formRef?.validate((valid) => {
    if (valid) {
      Object.keys(ruleForm).forEach((key) => {
        utools.dbStorage.setItem(key, Reflect.get(ruleForm, key));
      })
      open('配置更新成功~');
    }
  })
}

const uploadRequest = async (options: UploadRequestOptions) => {
  await client.put(`${Date.now()}-${options.file.name}`, options.file);
  open('文件上传成功~');
  await refresh();
}

onMounted(() => {
  init();
})
</script>