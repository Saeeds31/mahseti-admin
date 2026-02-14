<template>
  <b-container fluid class="py-4" v-if="checkPermission(['specifications_store'])">
    <b-row>
      <b-col cols="12" lg="8" class="mx-auto">
        <b-card header="ثبت ویژگی جدید">
          <b-form @submit.prevent="submitForm">
            <b-row>
              <!-- نام ویژگی -->
              <b-col cols="12" md="6">
                <b-form-group label="عنوان ویژگی" label-for="title">
                  <b-form-input id="title" v-model="form.title" placeholder="مثال: جنس، رنگ، سایز..." required />
                </b-form-group>
              </b-col>
            </b-row>

            <!-- لیست مقادیر -->
            <b-row class="mt-3">
              <b-col cols="12">
                <b-form-group label="مقادیر ویژگی">
                  <div v-for="(value, index) in form.values" :key="index" class="d-flex mb-2">
                    <b-form-input v-model="form.values[index]"
                      placeholder="مقدار ویژگی (مثال: آهنی، آلومینیومی، چوبی)" />
                    <b-button variant="danger" class="ms-2" @click="removeValue(index)">
                      حذف
                    </b-button>
                  </div>
                  <b-button variant="success" @click="addValue">افزودن مقدار</b-button>
                </b-form-group>
              </b-col>
            </b-row>

            <!-- دکمه ثبت -->
            <b-row>
              <b-col cols="12" class="text-end">
                <b-button :disabled="loading" variant="primary" type="submit">ثبت ویژگی</b-button>
              </b-col>
            </b-row>
          </b-form>
        </b-card>
      </b-col>
    </b-row>
  </b-container>
</template>

<script setup>
import { ref } from "vue"
import axios from "axios"
import { toast } from "vue3-toastify"
import "vue3-toastify/dist/index.css"
import { useRouter } from "vue-router"
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const router = useRouter();
const form = ref({
  title: "",
  values: [""], // حداقل یک مقدار خالی اول داشته باشه
})
let loading = ref(false);

const addValue = () => {
  form.value.values.push("")
}

const removeValue = (index) => {
  form.value.values.splice(index, 1)
}

const submitForm = async () => {
  loading.value = true;
  try {
    let formData = new FormData();
    formData.append("title", form.value.title)
    for (const value of form.value.values) {
      if (value && value.trim() !== "") {
        formData.append("values[]", value)
      }
    }
    const { data } = await axios.post("/specifications", formData)
    toast.success("ویژگی و مقادیر با موفقیت ثبت شدند")
    router.push("/products/specification")
    form.value = { title: "", values: [""] }
  } catch (e) {
    console.log(e)
    toast.error("خطا در ثبت ویژگی یا مقادیر")
  } finally {
    loading.value = false;
  }
}
</script>

<style scoped>
.ms-2 {
  margin-inline-start: 0.5rem;
}
</style>
