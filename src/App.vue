<script setup>
import { onMounted, reactive, ref, watch, computed } from "vue";
import { Plus, Delete } from "@element-plus/icons-vue";
const notes = ref([]);
const dialogVisible = ref(false);
const formRef = ref();
const form = ref({
  title: "",
  content: "",
});

const rules = reactive({
  title: [
    {
      required: true,
      message: "Please input title",
      trigger: "change",
    },
  ],
  content: [
    {
      required: true,
      message: "Please input content",
      trigger: "change",
    },
  ],
});

// Sorts items in ascending order by creation date
const notes_asc = computed(() =>
  notes.value.sort((a, b) => {
    return b.createdAt - a.createdAt;
  })
);

const showDialog = () => {
  dialogVisible.value = true;
  formRef.value.clearValidate();
};

const addNote = async (formEl) => {
  if (!formEl) return;
  await formEl.validate((valid) => {
    if (valid) {
      notes.value.push({
        title: form.value.title,
        content: form.value.content,
        createdAt: new Date().getTime(),
      });

      dialogVisible.value = false;
      resetForm();
    } else {
      return;
    }
  });
};

watch(
  notes,
  (newValue) => {
    localStorage.setItem("notes", JSON.stringify(newValue));
  },
  { deep: true }
);

const deleteNote = (index) => {
  notes.value.splice(index, 1);
};
const resetForm = () => {
  form.value.title = "";
  form.value.content = "";
};

onMounted(() => {
  notes.value = JSON.parse(localStorage.getItem("notes")) || [];
});
</script>

<template>
  <div>
    <!-- // MARK: Header -->
    <el-row align="middle">
      <el-col :span="12">
        <h1>My Notes</h1>
      </el-col>
      <el-col :span="12" align="right">
        <el-button
          type="primary"
          :icon="Plus"
          circle
          size="large"
          @click="showDialog"
        ></el-button>
      </el-col>
    </el-row>
    <br />
    <!-- //MARK: Card container -->
    <el-row :gutter="20" class="note-container">
      <el-col
        v-for="(note, index) in notes_asc"
        :key="note.title"
        :xs="24"
        :sm="12"
        :md="6"
      >
        <el-card
          shadow="always"
          :body-style="{ padding: '20px' }"
          class="box-card"
        >
          <template #header>
            <div class="card-header">
              <h3>{{ note.title }}</h3>
              <el-button
                type="danger"
                :icon="Delete"
                @click="deleteNote(index)"
              ></el-button>
            </div>
          </template>
          {{ note.content }}
        </el-card>
      </el-col>
    </el-row>
    <!-- //MARK: Dialog Form -->
    <el-dialog
      v-model="dialogVisible"
      width="80%"
      :close-on-click-modal="false"
    >
      <template #header>
        <h2>New Note</h2>
      </template>
      <span>
        <!-- //MARK: Form -->
        <el-form
          :model="form"
          ref="formRef"
          label-width="80px"
          label-position="top"
          :rules="rules"
        >
          <el-form-item label="Title" prop="title">
            <el-input
              v-model="form.title"
              placeholder="Type your title"
            ></el-input>
          </el-form-item>
          <el-form-item label="Content" prop="content">
            <el-input
              type="textarea"
              v-model="form.content"
              placeholder="Type your note content"
              rows="4"
            ></el-input>
          </el-form-item>
        </el-form>
      </span>
      <template #footer>
        <span>
          <el-button @click="dialogVisible = false">Cancel</el-button>
          <el-button type="primary" @click="addNote(formRef)">OK</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  position: relative;
  height: 100%;
  top: 0px;
  bottom: 0px;
}

.note-container .el-col {
  margin-bottom: 20px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.box-card {
  max-height: 350px;
  overflow: auto;
}
</style>
