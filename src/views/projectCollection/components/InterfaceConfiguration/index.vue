<template>
  <el-form ref="formRef" label-position="top" class="card interface-configuration">
    <el-row :gutter="20" style="margin-bottom: 20px">
      <el-col :span="20">
        <span class="text"> {{ props.apiTitle }} </span>
      </el-col>
      <el-col :span="4" class="right-aligned">
        <el-button type="info" @click="ElMessage.success('保存成功')">
          保存<el-icon class="el-icon--right"><MessageBox /></el-icon>
        </el-button>
      </el-col>
    </el-row>
    <el-row :gutter="20">
      <el-col :span="4">
        <el-form-item>
          <el-select placeholder="请选择请求方法" v-model="formData.requestMethod">
            <el-option label="GET" value="GET"></el-option>
            <el-option label="POST" value="POST"></el-option>
            <el-option label="DELETE" value="DELETE"></el-option>
            <el-option label="PUT" value="PUT"></el-option>
          </el-select>
        </el-form-item>
      </el-col>
      <el-col :span="16">
        <el-form-item>
          <el-input placeholder="请输入接口 URL" clearable v-model="formData.apiUrl">
            <template #prepend>Http://</template>
          </el-input>
        </el-form-item>
      </el-col>
      <el-col :span="4" class="right-aligned">
        <el-button type="primary" @click="console.log('发送数据:', formData)">
          发送<el-icon class="el-icon--right"><Connection /></el-icon>
        </el-button>
      </el-col>
    </el-row>
    <div class="interface-query card">
      <el-tabs v-model="activeQuery">
        <el-tab-pane label="Params" name="first">
          <div class="query-params">
            <QueryTable v-model:queryParams="formData.queryParams" />
          </div>
        </el-tab-pane>
        <el-tab-pane label="Auth" name="second">
          <div class="query-params">
            <el-form-item label="类型">
              <el-select placeholder="请选择请求类型" v-model="formData.authType">
                <el-option label="No Auth" value="noAuth"></el-option>
                <el-option label="从父级继承" value="father"></el-option>
              </el-select>
            </el-form-item>
          </div>
        </el-tab-pane>
        <el-tab-pane label="Headers" name="third">
          <div class="query-params">
            <QueryTable v-model:queryParams="formData.queryHeaders" />
          </div>
        </el-tab-pane>
        <el-tab-pane label="Body" name="fourth">
          <div class="query-params">
            <el-radio-group v-model="queryBody">
              <el-radio :label="1">none</el-radio>
              <el-radio :label="2">form-data</el-radio>
              <el-radio :label="3">x-www-form-urlencoded</el-radio>
              <el-radio :label="4">json</el-radio>
              <el-radio :label="5">xml</el-radio>
              <el-radio :label="6">raw</el-radio>
            </el-radio-group>
            <!-- 根据选中的 queryBody 显示不同的内容 -->
            <div class="query-body-container">
              <div v-if="queryBody === 1"><el-empty :image-size="70" /></div>
              <div v-if="queryBody === 2">
                <QueryTable v-model:queryParams="formData.queryBodyForm" />
              </div>
              <div v-if="queryBody === 3">
                <QueryTable v-model:queryParams="formData.queryBodyFormX" />
              </div>
              <div v-if="queryBody === 4">
                <el-input
                  v-model="queryJsonBody"
                  :autosize="{ minRows: 6, maxRows: 10 }"
                  type="textarea"
                  placeholder="Please input"
                />
              </div>
              <div v-if="queryBody === 5">
                <el-input
                  v-model="queryXmlBody"
                  :autosize="{ minRows: 6, maxRows: 10 }"
                  type="textarea"
                  placeholder="Please input"
                />
              </div>
              <div v-if="queryBody === 6">
                <el-input
                  v-model="queryRawBody"
                  :autosize="{ minRows: 6, maxRows: 10 }"
                  type="textarea"
                  placeholder="Please input"
                />
              </div>
            </div>
          </div>
        </el-tab-pane>
      </el-tabs>
    </div>
    <div class="interface-response card">
      <el-tabs v-model="activeResponse">
        <el-tab-pane label="Body" name="first">
          <div class="response-body">
            <div class="res-body-toolBar">
              <el-radio-group v-model="resBodyRadio" size="small">
                <el-radio-button label="Pretty" />
                <el-radio-button label="Raw" />
                <el-radio-button label="Preview" />
              </el-radio-group>
              <el-select v-model="ResSelect" size="small" v-if="resBodyRadio === 'Pretty'">
                <el-option v-for="item in ResFileOption" :key="item.value" :label="item.label" :value="item.value">
                  <span>{{ item.label }}</span>
                </el-option>
              </el-select>
            </div>
            <div v-if="responseBody == ''"><el-empty :image-size="70" /></div>
            <el-input v-else type="textarea" v-model="responseBody" :autosize="{ minRows: 6, maxRows: 10 }" disabled />
          </div>
        </el-tab-pane>
        <el-tab-pane label="Cookies" name="second"><div class="response-params">No Cookies</div></el-tab-pane>
        <el-tab-pane label="Headers" name="third">
          <div class="response-params">
            <el-table :data="resHeaders" border style="width: 100%">
              <el-table-column prop="key" label="key" />
              <el-table-column prop="value" label="Value" />
            </el-table>
          </div>
        </el-tab-pane>
      </el-tabs>
    </div>
  </el-form>
</template>

<script setup lang="ts">
import { reactive, ref } from "vue";
import QueryTable from "./QueryTable.vue";
import { ElMessage } from "element-plus";

const props = defineProps({
  apiTitle: String
});

//测试数据
const queryParams = [{ key: "param1", value: "value1", description: "描述1" }];
const queryHeaders = [{ key: "header1", value: "value1", description: "描述1" }];
const queryBodyForm = [{ key: "header1", value: "value1", description: "描述1" }];
const queryBodyFormX = [{ key: "header1", value: "value1", description: "描述1" }];

const formRef = ref(null);
const formData = reactive({
  requestMethod: "GET",
  apiUrl: "",
  authType: "noAuth",
  queryParams: queryParams,
  queryHeaders: queryHeaders,
  queryBodyForm: queryBodyForm,
  queryBodyFormX: queryBodyFormX,
  queryJsonBody: "",
  queryXmlBody: "",
  queryRawBody: ""
  // 其他表单属性
});

const activeQuery = ref("first");
const activeResponse = ref("first");
const resBodyRadio = ref("Pretty");
const ResSelect = ref("JSON");
const queryBody = ref(1);
const queryJsonBody = ref("");
const queryXmlBody = ref("");
const queryRawBody = ref("");
const responseBody = ref("test");

//响应体格式化显示配置
const ResFileOption = [
  {
    value: "JSON",
    label: "JSON"
  },
  {
    value: "XML",
    label: "XML"
  },
  {
    value: "HTML",
    label: "HTML"
  },
  {
    value: "TEXT",
    label: "TEXT"
  },
  {
    value: "AUTO",
    label: "AUTO"
  }
];

//响应头配置
const resHeaders = [
  {
    key: "Date",
    value: "Thu, 14 Sep 2023 09:24:49 GMT"
  },
  {
    key: "Server",
    value: "Werkzeug/2.3.3 Python/3.9.16"
  },
  {
    key: "Content-Type",
    value: "application/json"
  },
  {
    key: "Content-Length",
    value: "Tom"
  }
];
// 其他的设置和事件处理函数
</script>

<style scoped lang="scss">
@import "./index.scss";
</style>
