<template>
  <div id="viewQuestionView">
    <a-row :gutter="[24, 24]">
      <a-col :md="12" :xs="24">
        <a-tabs default-active-key="question">
          <a-tab-pane key="question" title="题目">
            <a-card v-if="question" :title="question.title">
              <a-descriptions
                title="判题条件"
                :column="{ xs: 1, md: 2, lg: 3 }"
              >
                <a-descriptions-item label="时间限制">
                  {{ question.judgeConfig.timeLimit ?? 0 }}
                </a-descriptions-item>
                <a-descriptions-item label="内存限制">
                  {{ question.judgeConfig.memoryLimit ?? 0 }}
                </a-descriptions-item>
                <a-descriptions-item label="堆栈限制">
                  {{ question.judgeConfig.stackLimit ?? 0 }}
                </a-descriptions-item>
              </a-descriptions>
              <MdViewer :value="question.content || ''" />
              <template #extra>
                <a-space wrap>
                  <a-tag
                    v-for="(tag, index) of question.tags"
                    :key="index"
                    color="green"
                    >{{ tag }}
                  </a-tag>
                </a-space>
              </template>
            </a-card>
          </a-tab-pane>
          <a-tab-pane key="comment" title="评论">
            <a-space>
              <a-input
                :style="{ width: '320px' }"
                placeholder="请输入您的评论"
                allow-clear
                size="large"
                v-model="comment"
              />
              <a-button type="primary" @click="newComment">发表评论</a-button>
            </a-space>
            <h1 :style="{ margin: '60px 0 40px 0' }">评论</h1>
            <div class="commentItem">
              <template v-for="item in commentList" :key="item">
                <a-comment
                  :content="item.comment"
                  :author="item.userVO.userName"
                  avatar="http://82.156.110.191:9099/avatar.png"
                  :datetime="item.userVO.createTime.split('T')[0]"
                >
                  <template
                    v-for="item2 in item.commentRelationVoList"
                    :key="item2"
                  >
                    <a-comment
                      :author="item2.userVO.userName"
                      avatar="http://82.156.110.191:9099/avatar2.png"
                      :content="item2.comment"
                      :datetime="item2.userVO.createTime.split('T')[0]"
                    >
                    </a-comment>
                  </template>
                </a-comment>
                <a-trigger
                  trigger="click"
                  :unmount-on-close="false"
                  :popup-translate="[200, -33.5]"
                >
                  <a-button
                    type="outline"
                    @click="reply(item.commentId)"
                    :style="{
                      margin: '20px 0 0 0',
                    }"
                    >回复
                  </a-button>
                  <template #content>
                    <div class="demo-basic">
                      <a-input
                        :style="{
                          width: '320px',
                        }"
                        placeholder="请输入您的回复"
                        allow-clear
                        size="large"
                        v-model="comment2"
                      />
                    </div>
                  </template>
                </a-trigger>
                <a-divider />
              </template>
            </div>
          </a-tab-pane>
          <a-tab-pane key="answer" title="答案"> 暂时无法查看答案</a-tab-pane>
        </a-tabs>
      </a-col>
      <a-col :md="12" :xs="24">
        <a-form :model="form" layout="inline">
          <a-form-item
            field="language"
            label="编程语言"
            style="min-width: 240px"
          >
            <a-select
              v-model="form.language"
              :style="{ width: '320px' }"
              placeholder="选择编程语言"
            >
              <a-option>java</a-option>
              <a-option>cpp</a-option>
              <a-option>go</a-option>
              <a-option>html</a-option>
            </a-select>
          </a-form-item>
        </a-form>
        <CodeEditor
          :value="form.code as string"
          :language="form.language"
          :handle-change="changeCode"
        />
        <a-divider size="0" />
        <a-button type="primary" style="min-width: 200px" @click="doSubmit">
          提交代码
        </a-button>
      </a-col>
    </a-row>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, watchEffect, withDefaults, defineProps } from "vue";
import message from "@arco-design/web-vue/es/message";
import CodeEditor from "@/components/CodeEditor.vue";
import MdViewer from "@/components/MdViewer.vue";
import axios from "axios";
import {
  QuestionControllerService,
  QuestionSubmitAddRequest,
  QuestionVO,
} from "../../../generated";

interface Props {
  id: string;
}

const props = withDefaults(defineProps<Props>(), {
  id: () => "",
});

const question = ref<QuestionVO>();

let comment = ref();
let newComment = async () => {
  if (!question.value?.id) {
    return;
  }
  const data = {
    comment: comment.value,
    questionId: question.value.id,
  };
  const config = {
    headers: {
      "Content-Type": "application/json",
      "Referrer-Policy": "unsafe-url",
    },
  };
  axios
    .post(
      "https://www.xianyuwang.online/api/comment/add/questioncomment",
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
};
const loadData = async () => {
  const res = await QuestionControllerService.getQuestionVoByIdUsingGet(
    props.id as any
  );
  if (res.code === 0) {
    question.value = res.data;
  } else {
    message.error("加载失败，" + res.message);
  }
};

let commentList = ref();
const loadComment = async () => {
  if (!question.value?.id) {
    return;
  }
  const data = {
    id: question.value.id,
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
      commentList.value = response.data.data;
      console.log("success");
    })
    .catch((error) => {
      console.log(error);
      console.log("error");
    });
};
let comment2 = ref();
let flag = ref(true);
const reply = async (id: any) => {
  if (flag.value == false) {
    flag.value = true;
  } else {
    flag.value = false;
    return;
  }
  const data = {
    comment: comment2.value,
    targetCommentId: id,
  };
  const config = {
    headers: {
      "Content-Type": "application/json",
      "Referrer-Policy": "unsafe-url",
    },
  };
  axios
    .post(
      "https://www.xianyuwang.online/api/comment/add/commentrelation",
      JSON.stringify(data),
      config
    )
    .then((response) => {
      console.log(response.data);
      console.log("success");
      message.success("评论成功");
      comment2.value = "";
      loadComment();
    })
    .catch((error) => {
      console.log(error);
      console.log("error");
    });
};

const form = ref<QuestionSubmitAddRequest>({
  language: "java",
  code: "",
});

/**
 * 提交代码
 */
const doSubmit = async () => {
  if (!question.value?.id) {
    return;
  }

  const res = await QuestionControllerService.doQuestionSubmitUsingPost({
    ...form.value,
    questionId: question.value.id,
  });
  if (res.code === 0) {
    message.success("提交成功");
  } else {
    message.error("提交失败," + res.message);
  }
};

/**
 * 页面加载时，请求数据
 */
onMounted(async () => {
  await loadData();
  await loadComment();
});

const changeCode = (value: string) => {
  form.value.code = value;
};
</script>

<style>
#viewQuestionView {
  max-width: 1400px;
  margin: 0 auto;
}

#viewQuestionView .arco-space-horizontal .arco-space-item {
  margin-bottom: 0 !important;
}
</style>
