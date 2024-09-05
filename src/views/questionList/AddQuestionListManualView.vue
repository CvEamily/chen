<template>
  <div id="addQuestionView">
    <h2>手工创建题单</h2>
    <a-form :model="form" label-align="left">
      <a-form-item field="title" label="标题">
        <a-input v-model="form.title" placeholder="请输入标题" />
      </a-form-item>
      <a-form-item field="tags" label="标签">
        <a-input-tag v-model="form.tags" placeholder="请选择标签" allow-clear />
      </a-form-item>
      <QuestionsView ref="questionSelect"></QuestionsView>
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
import QuestionsView from "@/components/QuestionsView.vue";
import axios from "axios";
const questionSelect = ref();
const questionIdList = ref();
const getList = () => {
  console.log(questionSelect.value);
  questionSelect.value.getList();
  questionIdList.value = questionSelect.value.list;
  console.log("questionIdList.value", questionIdList.value);
};
const route = useRoute();
// 如果页面地址包含 update，视为更新页面
const updatePage = route.path.includes("update");

let form = ref({
  title: "",
  tags: [],
  questionIdList: [],
});

/**
 * 根据题目 id 获取老的数据
 */
let id = ref();
const loadData = async () => {
  id.value = route.query.id;
  if (!id.value) {
    return;
  }
  const res = await QuestionControllerService.getQuestionByIdUsingGet(
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

onMounted(() => {
  loadData();
});
const doSubmit = async () => {
  getList();
  console.log(form.value);
  // 区分更新还是创建
  if (updatePage) {
    console.log("更新表单");
    const data = {
      id: id.value.id,
      questionIdList: form.value.questionIdList,
      tags: form.value.tags,
      title: form.value.title,
    };
    const config = {
      headers: {
        "Content-Type": "application/json",
        "Referrer-Policy": "unsafe-url",
      },
    };
    axios
      .post(
        "https://www.xianyuwang.online/api/comment/queryComment",
        JSON.stringify(data),
        config
      )
      .then((response) => {
        console.log(response.data);
        console.log("success");
      })
      .catch((error) => {
        console.log(error);
        console.log("error");
      });
    const res = await QuestionControllerService.updateQuestionUsingPost(
      form.value
    );
    if (res.code === 0) {
      message.success("更新成功");
    } else {
      message.error("更新失败，" + res.message);
    }
  } else {
    const res = await QuestionControllerService.addQuestionListUsingPost(
      form.value
    );
    if (res.code === 0) {
      message.success("创建成功");
      alert("创建成功");
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

<style scoped>
#addQuestionView {
}
</style>
