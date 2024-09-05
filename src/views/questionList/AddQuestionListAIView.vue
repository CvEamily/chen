<template>
  <div id="addQuestionListAIView">
    <h2>AI生成题单</h2>
    <a-form :model="form" label-align="left">
      <a-form-item field="title" label="标题">
        <a-input v-model="form.title" placeholder="请输入标题" />
      </a-form-item>
      <a-form-item field="tags" label="标签">
        <a-transfer
          v-if="show"
          show-search
          :data="tagList"
          :default-value="defaultValue"
          @change="handleChange"
          :source-input-search-props="{
            placeholder: 'source item search',
          }"
          :target-input-search-props="{
            placeholder: 'target item search',
          }"
          simple
        >
          <template
            #source-title="{ checked, indeterminate, onSelectAllChange }"
          >
            <div :style="styleHeader">
              标签库
              <a-checkbox
                :model-value="checked"
                :indeterminate="indeterminate"
                @change="onSelectAllChange"
                style="display: none"
              />
            </div>
          </template>
          <template #target-title="{ onClear }">
            <div :style="styleHeader">
              选择的标签
              <IconDelete @click="onClear" />
            </div>
          </template>
        </a-transfer>
      </a-form-item>
      <a-form-item field="tags" label="题目数量">
        <a-button type="text" status="success">{{ form.questionNum }}</a-button>
        <a-button type="primary" status="success" @click="changeQNumber(5)"
          >5</a-button
        >
        <a-button type="primary" status="success" @click="changeQNumber(10)"
          >10</a-button
        >
        <a-button type="primary" status="success" @click="changeQNumber(15)"
          >15</a-button
        >
        <a-button type="primary" status="success" @click="changeQNumber(20)"
          >20</a-button
        >
      </a-form-item>
      <a-form-item field="tags" label="热门程度">
        <a-rate grading allow-half @change="hanleRateChange" />
      </a-form-item>
      <a-form-item field="tags" label="题目难度比例">
        <div class="warp">
          <p>
            <span>简单</span>
            <span>:</span>
            <span>中等</span>
            <span>:</span>
            <span>困难</span>
          </p>
          <a-input-number
            v-model="form.difficultyRate[0]"
            style="width: 220px; margin-right: 10px"
            placeholder="Please Enter"
            class="input-demo"
            :min="1"
            :max="100"
          />
          <a-input-number
            v-model="form.difficultyRate[1]"
            style="width: 220px; margin-right: 10px"
            placeholder="Please Enter"
            class="input-demo"
            :min="1"
            :max="100"
          />
          <a-input-number
            v-model="form.difficultyRate[2]"
            style="width: 220px; margin-right: 10px"
            placeholder="Please Enter"
            class="input-demo"
            :min="1"
            :max="100"
          />
        </div>
      </a-form-item>
      <div style="margin-top: 16px" />
      <a-form-item>
        <a-button type="primary" style="min-width: 200px" @click="doSubmit"
          >提交
        </a-button>
      </a-form-item>
    </a-form>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";
import { QuestionControllerService } from "../../../generated";
import message from "@arco-design/web-vue/es/message";
import { useRoute } from "vue-router";
import axios from "axios";

const route = useRoute();
// 如果页面地址包含 update，视为更新页面
const updatePage = route.path.includes("update");

const changeQNumber = (number: any) => {
  form.value.questionNum = number;
};
let tagList: { value: string; label: string }[] = [];

const defaultValue = ["", "", ""];
let show = ref(false);
let rate = ref();
let form = ref({
  title: "",
  difficultyRate: [],
  needTags: [] as [],
  popScore: 0,
  questionNum: 0,
});

/**
 * 根据题目 id 获取老的数据
 */
const loadData = async () => {
  const id = route.query.id;
  if (!id) {
    return;
  }
  const res = await QuestionControllerService.getQuestionListByIdUsingGet(
    id as any
  );
  if (res.code === 0) {
    form.value = res.data as any;
    // json 转 js 对象
    if (!form.value.judgeCase) {
      form.value.judgeCase = [
        {
          input: "",
          output: "",
        },
      ];
    } else {
      form.value.judgeCase = JSON.parse(form.value.judgeCase as any);
    }
    if (!form.value.judgeConfig) {
      form.value.judgeConfig = {
        memoryLimit: 1000,
        stackLimit: 1000,
        timeLimit: 1000,
      };
    } else {
      form.value.judgeConfig = JSON.parse(form.value.judgeConfig as any);
    }
    if (!form.value.tags) {
      form.value.tags = [];
    } else {
      form.value.tags = JSON.parse(form.value.tags as any);
    }
  } else {
    message.error("加载失败，" + res.message);
  }
};
const getTag = async () => {
  // interface res {
  //   data: typeof array;
  // }
  const res: any = await QuestionControllerService.getTag();
  console.log(res.data);
  let array = res.data;
  array.forEach((element: any) => {
    let obj = {
      value: "",
      label: "",
    };
    obj.value = element;
    obj.label = element;
    tagList.push(obj);
  });
  console.log(tagList);
  console.log(show.value);
  show.value = true;
};
onMounted(() => {
  loadData();
  getTag();
});
const handleChange = (a: any) => {
  form.value.needTags = a.slice(3);
};
const hanleRateChange = (a: any) => {
  form.value.popScore = a;
};
const doSubmit = async () => {
  if (updatePage) {
    console.log("更新题目");
    const res = await QuestionControllerService.updateQuestionListUsingPost(
      form.value
    );
    if (res.code === 0) {
      message.success("更新成功");
    } else {
      message.error("更新失败，" + res.message);
    }
  } else {
    const res = await QuestionControllerService.addQuestionListAIUsingPost(
      form.value
    );
    if (res.code === 0) {
      message.success("创建成功");
    } else {
      message.error("创建失败，" + res.message);
    }
  }
};

/**
 * 新增判题用例
 */
const handleAdd = () => {
  form.value.judgeCase.push({
    input: "",
    output: "",
  });
};

/**
 * 删除判题用例
 */
const handleDelete = (index: number) => {
  form.value.judgeCase.splice(index, 1);
};

const onContentChange = (value: string) => {
  form.value.content = value;
};

const onAnswerChange = (value: string) => {
  form.value.answer = value;
};
</script>

<style scoped lang="scss">
.warp {
  p {
    display: flex;
    span:nth-child(2n-1) {
      display: block;
      width: 220px;
      text-align: center;
    }
    span:nth-child(2n) {
      margin: 0 3.5px;
    }
  }
}
#addQuestionListAIView {
  button {
    margin-right: 2rem;
  }
}
</style>
