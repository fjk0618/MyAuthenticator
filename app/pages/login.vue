<script setup lang="ts">
import type { FormErrorEvent, FormSubmitEvent } from "#ui/types";
import { toast } from "@steveyuowo/vue-hot-toast";

const { fetch: refreshSesion } = useUserSession();

const { authenticate } = useWebAuthn();

const state = reactive({
  username: "",
  password: "",
});

const isLoading = ref(false);

const loginWithPasskey = async () => {
  await authenticate()
    .then(async (res) => {
      if (!res) return;
      await refreshSesion();
      navigateTo("/");
      toast.success("Login successful");
    })
    .catch((err) => {
      console.error(err);
      toast.error(err?.data?.message ?? err);
    });
};

const onSubmit = async (event: FormSubmitEvent<Login>) => {
  isLoading.value = true;
  const toastid = toast.loading("loading...");
  await $fetch("/api/auth/login", {
    method: "POST",
    body: event.data,
  })
    .then(async (res) => {
      await refreshSesion();
      navigateTo("/");
      toast.update(toastid, {
        message: res.message,
        type: "success",
      });
      isLoading.value = false;
    })
    .catch((err) => {
      toast.update(toastid, {
        message: err?.data?.message ?? err,
        type: "error",
      });
      isLoading.value = false;
      console.error(err);
    });
};

async function onError(event: FormErrorEvent) {
  console.log(event.errors[0]);
  toast.error(event.errors[0]?.message!);
}
</script>

<template>
  <div class="flex-center h-dvh overflow-hidden">
    <UCard>
      <UForm
        :schema="loginSchema"
        :state="state"
        class="flex flex-col items-center mb-2"
        @submit="onSubmit"
        @error="onError"
      >
        <span className="text-lg xs:text-xl font-semibold uppercase mb-4 mt-2"
          >2FA管理系统</span
        >
        <UFormField label="用户名" size="lg">
          <UInput
            v-model="state.username"
            placeholder="请输入用户名"
            icon="i-heroicons-user-circle"
            name="username"
            size="lg"
            required
          />
        </UFormField>
        <UFormField label="密码" size="lg">
          <UInput
            v-model="state.password"
            placeholder="请输入密码"
            icon="i-heroicons-key"
            name="password"
            type="password"
            size="lg"
            required
          />
        </UFormField>
        <div class="flex flex-center p-2 relative">
          <UButton :disabled="isLoading" type="submit" variant="soft" size="lg"
            >登录</UButton
          >
        </div>
        <USeparator
          label="OR"
          color="neutral"
          size="sm"
          :ui="{ root: 'my-4', label: 'text-[10px]' }"
        />
        <UButton
          icon="i-carbon-fingerprint-recognition"
          variant="soft"
          color="primary"
          size="lg"
          @click="loginWithPasskey"
          >使用Passkey登录</UButton
        >
      </UForm>
    </UCard>
  </div>
</template>
